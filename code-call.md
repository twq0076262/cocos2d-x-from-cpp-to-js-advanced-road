# [【cocos2d-x 从 c++ 到 js】09：JS 与 C++ 的交互 1——JS 代码调用 C++ 代码](http://goldlion.blog.51cto.com/4127613/1353583)

之前我们讲过，在游戏启动时，我们要通过 SpiderMonkey 引擎的注册接口，向 SpiderMonkey 注册相应的从 C++ 到 JS 的绑定函数，这些函数用于把 JS 函数调用代码转换成对应 C++ 函数调用来执行。

```
//在AppDelegate::applicationDidFinishLaunching函数中
    ScriptingCore* sc = ScriptingCore::getInstance();
    sc->addRegisterCallback(register_all_cocos2dx);
    sc->addRegisterCallback(register_all_cocos2dx_extension);
    sc->addRegisterCallback(register_cocos2dx_js_extensions);
    sc->addRegisterCallback(register_all_cocos2dx_extension_manual);
    sc->addRegisterCallback(jsb_register_chipmunk);
    sc->addRegisterCallback(JSB_register_opengl);
    sc->addRegisterCallback(jsb_register_system);
    sc->addRegisterCallback(MinXmlHttpRequest::_js_register);
    sc->addRegisterCallback(register_jsb_websocket);
    sc->addRegisterCallback(register_all_cocos2dx_builder);
    sc->addRegisterCallback(register_CCBuilderReader);
    sc->addRegisterCallback(register_all_cocos2dx_gui);
    sc->addRegisterCallback(register_all_cocos2dx_gui_manual);
    sc->addRegisterCallback(register_all_cocos2dx_studio);
    sc->addRegisterCallback(register_all_cocos2dx_studio_manual);
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      
    sc->addRegisterCallback(register_all_cocos2dx_spine);
```

可以看到上面导入了 Cocos2d-x 的各种库，核心库，扩展，opengl，物理引擎，websocket，CCB 等等等等。

下面我们说 JS 代码如何调用 C++ 代码。
首先，在创建 JS 对象的时候，也会创建一个对应的 C++ 对象。换句话说，JS 对象是和 C++ 对象一一对应的（当然必须是引擎支持的，而且绑定了接口的）。然后，在 JS 对象执行函数时，发生了什么呢？  SpiderMonkey 引擎会通过注册的接口，找到对应的 C++ 对象，调用该对象上对应的 C++ 函数。

换句话说，如果有下面的 JS 代码:

```
var node = cc.Node.create();
node.setVisible(false);
```

那么经过 SpiderMonkey 执行后，会调用下面的代码：

```
auto node = CCNode::create();
node->setVisible(false);
```

当然，SpiderMonkey 远远还不止干了这些，还做了很多事，比如绑定和查找 JS 和 C++ 对象的对应关系，包装参数为对应类型，类型安全检查，返回值包装等等。要知道他干了些什么，直接看引擎代码是更好的选择。

在 Cocos2d-x 3.0 版的引擎中，引擎目录结构进行了大规模重构。

![](images/9.jpg)

两个脚本语言被放到一个类似的目录中。其中 auto-generated/js-bindings 文件夹是 gxx-generator 工具自动生成的所有 C++ 绑定 JS 代码。而 javascript/bingdings 文件夹是手写的绑定代码，因为工具无法做到完全自动绑定，所以必须有一部分手写的（脚本语言都是这样，习惯就好了，谢谢）。

好，我们继续找刚才说的源代码。打开 jsb_cocos2dx_auto.cpp

```
JSBool js_cocos2dx_Node_create(JSContext *cx, uint32_t argc, jsval *vp)
{
    if (argc == 0) {
        cocos2d::Node* ret = cocos2d::Node::create();
        jsval jsret = JSVAL_NULL;
        do {
        if (ret) {
            js_proxy_t *proxy = js_get_or_create_proxy<cocos2d::Node>(cx, (cocos2d::Node*)ret);
            jsret = OBJECT_TO_JSVAL(proxy->obj);
        } else {
            jsret = JSVAL_NULL;
        }
    } while (0);
        JS_SET_RVAL(cx, vp, jsret);
        return JS_TRUE;
    }
    JS_ReportError(cx, "js_cocos2dx_Node_create : wrong number of arguments");
    return JS_FALSE;
}

```

这就是 cc.Node.create() 执行时，底层 C++ 跑的代码。所有的通过 JS 调用 C++ 的代码都与这个形式非常一致，首先看函数接口：  
第一个参数 JSContext *cx 是 JS 的上下文  
第二个参数 uint32_t argc 是 JS 代码中的参数个数，在这个里argc==0   
第三个参数 jsval *v p是 JS 代码中的具体参数  

继续分析

```
cocos2d::Node* ret = cocos2d::Node::create();
```

这个代码再熟悉不过了，标准的 Cocos2d-x 静态工场生成对象的代码

```
jsval jsret = JSVAL_NULL;
```

jsval jsret 是这个函数的返回值，这是表示的是一个 JS 对象

```
js_proxy_t *proxy = js_get_or_create_proxy<cocos2d::Node>(cx, (cocos2d::Node*)ret);
jsret = OBJECT_TO_JSVAL(proxy->obj);
```

注意这个模板函数，get_or_create，这就是把 JS 对象和 C++ 对象绑到一起的函数。他非常重要，注意 JS 和 C++ 对象是一一对应关系，理解这个特效，有助于我们利用 JS 语言的动态性进行更方便的编程。绑完之后，下面那个函数是用于获得返回值。

最后，函数都要返回一个 JSBool，表面这个函数执行是否成功。如果返回 JS_FALSE，还会通过 JS_ReportError 打印一条报错信息。注意！脚本语言有一个特点，如果函数运行失败了，则该函数后面的函数（在同一作用域中的）都会跳过执行。

继续看下一个函数

```
JSBool js_cocos2dx_Node_setVisible(JSContext *cx, uint32_t argc, jsval *vp)
{
    jsval *argv = JS_ARGV(cx, vp);
    JSBool ok = JS_TRUE;
    JSObject *obj = JS_THIS_OBJECT(cx, vp);
    js_proxy_t *proxy = jsb_get_js_proxy(obj);
    cocos2d::Node* cobj = (cocos2d::Node *)(proxy ? proxy->ptr : NULL);
    JSB_PRECONDITION2( cobj, cx, JS_FALSE, "js_cocos2dx_Node_setVisible : Invalid Native Object");
    if (argc == 1) {
        JSBool arg0;
        ok &= JS_ValueToBoolean(cx, argv[0], &arg0);
        JSB_PRECONDITION2(ok, cx, JS_FALSE, "js_cocos2dx_Node_setVisible : Error processing arguments");
        cobj->setVisible(arg0);
        JS_SET_RVAL(cx, vp, JSVAL_VOID);
        return JS_TRUE;
    }
    JS_ReportError(cx, "js_cocos2dx_Node_setVisible : wrong number of arguments: %d, was expecting %d", argc, 1);
    return JS_FALSE;
}
```

这个函数和前一个函数的区别是，这个函数有参数，并且他是一个类成员函数（上一个是类静态函数），所以，这里要有 this 指针。

```
jsval *argv = JS_ARGV(cx, vp);
JSBool ok = JS_TRUE;
JSObject *obj = JS_THIS_OBJECT(cx, vp);
js_proxy_t *proxy = jsb_get_js_proxy(obj);
cocos2d::Node* cobj = (cocos2d::Node *)(proxy ? proxy->ptr : NULL);
JSB_PRECONDITION2( cobj, cx, JS_FALSE, "js_cocos2dx_Node_setVisible : Invalid Native Object");
```

这一大段函数都在找那个 this 指针。注意，这里面有一个 Cocos2d-x 引擎经常出现的错误提示 Invalid Native Object。底层 C++ 对象被回收了，所以找不到了。

```
if (argc == 1) {
    JSBool arg0;
    ok &= JS_ValueToBoolean(cx, argv[0], &arg0);
    JSB_PRECONDITION2(ok, cx, JS_FALSE, "js_cocos2dx_Node_setVisible : Error processing arguments");
    cobj->setVisible(arg0);
    JS_SET_RVAL(cx, vp, JSVAL_VOID);
    return JS_TRUE;
}
```

CCNode::setVisible(xx) 只有一个参数，所以先判断 JS 的参数个数为1。JS_ValueToBoolean 完成 JS 对象到 C++ 对象的转换，注意！这是基本类型的转换，和查找对应的对象指针不同。你在 gxx-generator 生成的代码中会看到大量的这种转换。每次转换都要进行结果判断，如果失败，就打印错误信息。后面是直接调用对应 C++ 对象的 setVisible，以及设置返回值。

很繁琐不是吗？如果这种代码全部手写是不是会死人呢。肯定的吧。所以这些代码都是用脚本生成器做出来的（绝大部分）。


后面我们会继续讲解各种 JS 的绑定代码。

