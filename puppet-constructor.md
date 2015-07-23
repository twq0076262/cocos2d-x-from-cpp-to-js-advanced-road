# [【cocos2d-x 从 c++ 到 js】15：傀儡构造函数](http://goldlion.blog.51cto.com/4127613/1357886)

上篇我们以 Sprite 为例，分析了注册函数。但其中我们似乎遗漏了一个地方，那就是构造函数。因为 Cocos2d-x 在 C++ 层使用的是工场函数来生成对象，而不是构造函数。所以在 JS 层代码中，也需要有相应的对应机制来处理这件事。

看一下 jsb_cocos2dx_auto.hpp

```
extern JSClass  *jsb_cocos2d_Sprite_class;
extern JSObject *jsb_cocos2d_Sprite_prototype;
JSBool js_cocos2dx_Sprite_constructor(JSContext *cx, uint32_t argc, jsval *vp);
void js_cocos2dx_Sprite_finalize(JSContext *cx, JSObject *obj);
void js_register_cocos2dx_Sprite(JSContext *cx, JSObject *global);
void register_all_cocos2dx(JSContext* cx, JSObject* obj);
```

这声明了几个重要的对象和函数。JSClass 对象和原型对象、注册函数、自己实现的 finalize 的 Stub 等。但是我们发现js_cocos2dx_Sprite_constructor 构造函数并没有对应的实现代码，仅仅是一个声明而已。

需要注意的是，根据 JS 的原型继承，我们在生成 jsb_cocos2d_Sprite_prototype 原型时，需要传入一个构造函数，而构造函数 js_cocos2dx_Sprite_constructor 又是未实现的，那么他是如何做到的呢？

在 js_register_cocos2dx_Sprite 函数中查看生成 jsb_cocos2d_Sprite_prototype 原型的代码：

```
jsb_cocos2d_Sprite_prototype = JS_InitClass(
    cx, global,
    jsb_cocos2d_Node_prototype,
    jsb_cocos2d_Sprite_class,
    dummy_constructor<cocos2d::Sprite>, 0, // no constructor
    properties,
    funcs,
    NULL, // no static properties
    st_funcs);
```

注意到第五个参数是一个模板函数  dummy_constructor<cocos2d::Sprite>，字面意思是傀儡构造函数。

看一下这个模板函数的定义

```
template<class T>
static JSBool dummy_constructor(JSContext *cx, uint32_t argc, jsval *vp) {
    JS::RootedValue initializing(cx);
    JSBool isNewValid = JS_TRUE;
    JSObject* global = ScriptingCore::getInstance()->getGlobalObject();
    isNewValid = JS_GetProperty(cx, global, "initializing", &initializing) && JSVAL_TO_BOOLEAN(initializing);
    if (isNewValid)
    {
        TypeTest<T> t;
        js_type_class_t *typeClass = nullptr;
        std::string typeName = t.s_name();
        auto typeMapIter = _js_global_type_map.find(typeName);
        CCASSERT(typeMapIter != _js_global_type_map.end(), "Can't find the class type!");
        typeClass = typeMapIter->second;
        CCASSERT(typeClass, "The value is null.");
        JSObject *_tmp = JS_NewObject(cx, typeClass->jsclass, typeClass->proto, typeClass->parentProto);
        JS_SET_RVAL(cx, vp, OBJECT_TO_JSVAL(_tmp));
        return JS_TRUE;
    }
    JS_ReportError(cx, "Don't use `new cc.XXX`, please use `cc.XXX.create` instead! ");
    return JS_FALSE;
}
```

这个函数首先使用了 JS::RootedValue 类型的量来判断 GlobalObject 对象是否初始化完毕。JS::RootedValue 具体的原理暂时不用深究，你只需要知道这是 SpiderMonkey 引擎的一种内存管理方式即可。

然后使用了一个非常有趣的技巧，用一个模板类 TypeTest<T> t，取出对应的类型名。这是一个很不错的写法，能够不破坏函数签名，使得函数能够匹配 JS_InitClass 的参数类型，又能够在不同的上下文中里面获得需要的信息。我们看一下 TypeTest 的实现，这种写法在很多时候有很大的借鉴意义！

```
template< typename DERIVED >
class TypeTest
{
public:
    static const char* s_name()
    {
        // return id unique for DERIVED
        // ALWAYS VALID BUT STRING, NOT INT - BUT VALID AND CROSS-PLATFORM/CROSS-VERSION COMPATBLE
        // AS FAR AS YOU KEEP THE CLASS NAME
        return typeid( DERIVED ).name();
    }
};
```

最后我们在 _js_global_type_map 里查询对应的类型，取出相应的参数来调用 JS_NewObject 函数，生成对应的对象并设置为返回值。

