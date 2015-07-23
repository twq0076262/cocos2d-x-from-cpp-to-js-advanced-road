# [【cocos2d-x 从 c++ 到 js】10：JS 与 C++ 的交互 2——JS 与 C++ 的“函数重载”问题](http://goldlion.blog.51cto.com/4127613/1354185)

对于 C++ 来说，存在函数重载，例如：

```
void CCNode::setScale(float scale)
void CCNode::setScale(float scaleX,float scaleY)
```

这两个函数的函数名是一样的，但是参数表不同。最终在编译器编译后的函数签名不一样。

但是在 JavaScript 中并没有这种机制。怎么破？存在两种情况：

第一种、JS 需要调用重载的 C++ 函数接口
我们就以上面的函数为例，来看看在 cxx-generator 的自动生成代码中，函数重载是如何处理的。打开 jsb_cocos2dx_auto.cpp，找到如下代码：

```
JSBool js_cocos2dx_Node_setScale(JSContext *cx, uint32_t argc, jsval *vp)
{
    jsval *argv = JS_ARGV(cx, vp);
    JSBool ok = JS_TRUE;
    JSObject *obj = NULL;
    cocos2d::Node* cobj = NULL;
    obj = JS_THIS_OBJECT(cx, vp);
    js_proxy_t *proxy = jsb_get_js_proxy(obj);
    cobj = (cocos2d::Node *)(proxy ? proxy->ptr : NULL);
    JSB_PRECONDITION2( cobj, cx, JS_FALSE, "js_cocos2dx_Node_setScale : Invalid Native Object");
    do {
        if (argc == 2) {
            double arg0;
            ok &= JS_ValueToNumber(cx, argv[0], &arg0);
            if (!ok) { ok = JS_TRUE; break; }
            double arg1;
            ok &= JS_ValueToNumber(cx, argv[1], &arg1);
            if (!ok) { ok = JS_TRUE; break; }
            cobj->setScale(arg0, arg1);
            JS_SET_RVAL(cx, vp, JSVAL_VOID);
            return JS_TRUE;
        }
    } while(0);
    do {
        if (argc == 1) {
            double arg0;
            ok &= JS_ValueToNumber(cx, argv[0], &arg0);
            if (!ok) { ok = JS_TRUE; break; }
            cobj->setScale(arg0);
            JS_SET_RVAL(cx, vp, JSVAL_VOID);
            return JS_TRUE;
        }
    } while(0);
    JS_ReportError(cx, "js_cocos2dx_Node_setScale : wrong number of arguments");
    return JS_FALSE;
}
```

只是通过 argc 参数简单判断了一下参数个数，然后就执行对应的分支代码就好了。但是如果遇到参数个数相同，而类型不同的情况呢？尚不得而知。

第二种、不需要调用 C++ 函数接口，直接在 JS 层代码中模拟一下函数重载。这个就要利用 JS 语言的一些特性了。我们直接看 Cocos2d-html5 中的对应代码。哦，no，因为 html5 里面关于 CCNode::setScale 函数写了一点杂技代码。所以我们改成看 setPosition 函数吧。也是一样的。

```
setPosition:function (newPosOrxValue, yValue) {
    var locPosition = this._position;
    if (arguments.length == 2) {
        locPosition._x = newPosOrxValue;
        locPosition._y = yValue;
    } else if (arguments.length == 1) {
        locPosition._x = newPosOrxValue.x;
        locPosition._y = newPosOrxValue.y;
    }
    this.setNodeDirty();
},
```

可以看到，该代码使用了 JS 的 arguments 来判断参数个数，然后执行对应的分支代码。

好了，重载就说道这里，下篇继续~