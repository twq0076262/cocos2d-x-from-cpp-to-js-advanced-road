# [【cocos2d-x 从 c++ 到 js】13：回调函数 2——JSCallbackWrapper](http://goldlion.blog.51cto.com/4127613/1354903)

上一篇我们讲了按键回调，这一次我们来说说各种逻辑上的回调函数。

Cocos2d-x 里面一共有三大类回调函数，第一是按键回调 CCMenu 相关的，第二类是定时器相关的回调 
Schedule，第三类是 Action 相关的回调 CallFunc。这些回调从最初的引擎版本中就存在着，一直到现在。

一、绑定代码

在 JSB 的解决方案中，对于后两类函数，引擎统一封装成 JSCallbackWrapper 及其子类。

```
class JSCallbackWrapper: public cocos2d::Object {
public:
    JSCallbackWrapper();
    virtual ~JSCallbackWrapper();
    void setJSCallbackFunc(jsval obj);
    void setJSCallbackThis(jsval thisObj);
    void setJSExtraData(jsval data);
                                                                                                                                                                                                                                                                                                                                                                                                                  
    const jsval& getJSCallbackFunc() const;
    const jsval& getJSCallbackThis() const;
    const jsval& getJSExtraData() const;
protected:
    jsval _jsCallback;
    jsval _jsThisObj;
    jsval _extraData;
};
```

JSCallbackWrapper 从名字就可以知道，是 JS 回调函数的包装器。三个接口也一目了然，回调函数，this，外部数据。

```
// cc.CallFunc.create( func, this, [data])
// cc.CallFunc.create( func )
static JSBool js_callFunc(JSContext *cx, uint32_t argc, jsval *vp)
{
    if (argc >= 1 && argc <= 3) {
        jsval *argv = JS_ARGV(cx, vp);
        std::shared_ptr<JSCallbackWrapper> tmpCobj(new JSCallbackWrapper());
                                                                                                                                                                                                                                                                                                                                                                             
        tmpCobj->setJSCallbackFunc(argv[0]);
        if(argc >= 2) {
            tmpCobj->setJSCallbackThis(argv[1]);
        } if(argc == 3) {
            tmpCobj->setJSExtraData(argv[2]);
        }
                                                                                                                                                                                                                                                                                                                                                                             
        CallFuncN *ret = CallFuncN::create([=](Node* sender){
            const jsval& jsvalThis = tmpCobj->getJSCallbackThis();
            const jsval& jsvalCallback = tmpCobj->getJSCallbackFunc();
            const jsval& jsvalExtraData = tmpCobj->getJSExtraData();
                                                                                                                                                                                                                                                                                                                                                                                 
            bool hasExtraData = !JSVAL_IS_VOID(jsvalExtraData);
            JSObject* thisObj = JSVAL_IS_VOID(jsvalThis) ? nullptr : JSVAL_TO_OBJECT(jsvalThis);
                                                                                                                                                                                                                                                                                                                                                                                 
            JSB_AUTOCOMPARTMENT_WITH_GLOBAL_OBJCET
                                                                                                                                                                                                                                                                                                                                                                                 
            js_proxy_t *proxy = js_get_or_create_proxy<cocos2d::Node>(cx, sender);
                                                                                                                                                                                                                                                                                                                                                                                 
            jsval retval;
            if(jsvalCallback != JSVAL_VOID)
            {
                if (hasExtraData)
                {
                    jsval valArr[2];
                    valArr[0] = OBJECT_TO_JSVAL(proxy->obj);
                    valArr[1] = jsvalExtraData;
                                                                                                                                                                                                                                                                                                                                                                                         
                    JS_AddValueRoot(cx, valArr);
                    JS_CallFunctionValue(cx, thisObj, jsvalCallback, 2, valArr, &retval);
                    JS_RemoveValueRoot(cx, valArr);
                }
                else
                {
                    jsval senderVal = OBJECT_TO_JSVAL(proxy->obj);
                    JS_AddValueRoot(cx, &senderVal);
                    JS_CallFunctionValue(cx, thisObj, jsvalCallback, 1, &senderVal, &retval);
                    JS_RemoveValueRoot(cx, &senderVal);
                }
            }
                                                                                                                                                                                                                                                                                                                                                                                 
            // I think the JSCallFuncWrapper isn't needed.
            // Since an action will be run by a cc.Node, it will be released at the Node::cleanup.
            // By James Chen
            // JSCallFuncWrapper::setTargetForNativeNode(node, (JSCallFuncWrapper *)this);
        });
                                                                                                                                                                                                                                                                                                                                                                             
        js_proxy_t *proxy = js_get_or_create_proxy<cocos2d::CallFunc>(cx, ret);
        JS_SET_RVAL(cx, vp, OBJECT_TO_JSVAL(proxy->obj));
                                                                                                                                                                                                                                                                                                                                                                             
        JS_SetReservedSlot(proxy->obj, 0, argv[0]);
        if(argc > 1) {
            JS_SetReservedSlot(proxy->obj, 1, argv[1]);
        }
//        if(argc == 3) {
//            JS_SetReservedSlot(proxy->obj, 2, argv[2]);
//        }
                                                                                                                                                                                                                                                                                                                                                                             
      //  test->execute();
        return JS_TRUE;
    }
    JS_ReportError(cx, "Invalid number of arguments");
    return JS_FALSE;
}
```

这是 JS 层调用 cc.CallFunc.create 时，底层执行的 C++ 函数，这里面用了一些 C++11 的特性，包括 std::shared_ptr 智能指针和 lambda 表达式（也很简单，不熟悉的童鞋可以自己找资料熟悉下）。

这里面回调函数被封装到了lambda表达式里面，通过=方式引用外部的 tmpCobj 变量，这种方式跟 JS 的闭包非常类似。依然使用 JS_CallFunctionValue 进行函数调用。注意，这种调用方式跟 JS 里面的 apply 方式是很类似的。

这里面有一对函数非常有趣，JS_AddValueRoot 和JS_RemoveValueRoot，这两个函数 JS_CallFunctionValue 调用包起来了。因为这个 valArr 或 senderVal 是在栈上临时生成的，没有指定对应的 root。但是中间又进行了 JS 函数的调用，所以这两个值可能在  JS 函数调用的时候被 SpiderMonkey 虚拟机给垃圾回收掉（可以去看看 JS 的垃圾回收机制原理）。于是我们需要给他们挂一个 root，保护一下，不被回收掉。

二、调用代码

先看一下构造函数

```
CallFuncN * CallFuncN::create(const std::function<void(Node*)> &func)
{
    auto ret = new CallFuncN();
    if (ret && ret->initWithFunction(func) ) {
        ret->autorelease();
        return ret;
    }
    CC_SAFE_DELETE(ret);
    return nullptr;
}
```

```
bool CallFuncN::initWithFunction(const std::function<void (Node *)> &func)
{
    _functionN = func;
    return true;
}
```

传进来的 lambda 表达式被存为一个 std::function<void(Node*)>  类型。

调用代码异常简单，使用 _functionN 进行调用即可。

```
void CallFuncN::execute() {
    if (_callFuncN) {
        (_selectorTarget->*_callFuncN)(_target);
    }
    else if (_functionN) {
        _functionN(_target);
    }
}
```

对比上一篇中的方式，我认为这种调用方式更加合理，因为这种调用方式，对 C++ 层 Core 代码，隐藏了脚本机制。而之前的调用方式是显示通过脚本引擎来调用的。
看完此篇和前篇，我们仔细分析了 Cocos2d-x JSB 里面的回调函数的写法，详细对于读者而言自己实现一个回调函数已经不是什么特别困难的事情。

在刚完成此篇的时候，突然发现有这么一个帖子，讲的也是 JSB 回调函数，写得很不错，还是 IAP 的，可以作为额外阅读参考：

[Cocos2d-x 使用 iOS 游戏内付费 IAP(JSB 篇)](http://www.ityran.com/archives/5571)

还有一篇可以学习的：  
[JS 的回调函数的参数构造注记——Web 学习笔记（十八）](http://jiajing.elastos.org/2013/05/22/js%E7%9A%84%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0%E7%9A%84%E5%8F%82%E6%95%B0%E6%9E%84%E9%80%A0%E6%B3%A8%E8%AE%B0-web%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%EF%BC%88%E5%8D%81%E5%85%AB/)

关于回调函数的问题，先说这些吧。

下篇继续，我们来讨论一下注册函数的事