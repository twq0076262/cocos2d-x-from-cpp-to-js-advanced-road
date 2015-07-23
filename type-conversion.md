# [【cocos2d-x 从 c++ 到 js】11：JS 与 C++ 的交互 3——C++ 和 JS 类型转换](http://goldlion.blog.51cto.com/4127613/1354628)

在 SpiderMonkey 执行时，经常要把 JS 中的数据类型转换成 C++ 类型，比如 int，unit，string，各种容器等等。转换之后，才能够给对应的 C++ 函数传递参数，来完成对应的 C++ 函数的调用。反过来也是一样， C++ 的数据类型要返回到 JS 里面，这样 JS 层的代码才能继续跑，也需要把 C++ 类型转换为 JS 类型。

这些“基本数据类型”的转换，是通过预先编写的代码来完成的，cxx-generator 在生成自动 binding 的代码时，如果遇到类型转换，会调用这些写好的这些类型转换代码。了解这些预先写好的类型转换代码是十分必要的，因为有的时候需要调试 bug，有的时候会有一些不能识别的类型，要手写转换代码。

在 Cocos2d-x 引擎中，这些类型转换代码是存放在一个js_manual_conversions.h 和 js_manual_conversions.cpp 的文件中的。

我们看一下头文件，为了便于查看我删掉了模板函数的实现代码。

```
JSBool jsval_to_opaque( JSContext *cx, jsval vp, void **out );
JSBool jsval_to_int( JSContext *cx, jsval vp, int *out);
JSBool jsval_to_uint( JSContext *cx, jsval vp, unsigned int *out);
JSBool jsval_to_c_class( JSContext *cx, jsval vp, void **out_native, struct jsb_c_proxy_s **out_proxy);
/** converts a jsval (JS string) into a char */
JSBool jsval_to_charptr( JSContext *cx, jsval vp, const char **out);
jsval opaque_to_jsval( JSContext *cx, void* opaque);
jsval c_class_to_jsval( JSContext *cx, void* handle, JSObject* object, JSClass *klass, const char* class_name);
/* Converts a char ptr into a jsval (using JS string) */
jsval charptr_to_jsval( JSContext *cx, const char *str);
JSBool JSB_jsval_typedarray_to_dataptr( JSContext *cx, jsval vp, GLsizei *count, void **data, JSArrayBufferViewType t);
JSBool JSB_get_arraybufferview_dataptr( JSContext *cx, jsval vp, GLsizei *count, GLvoid **data );
// some utility functions
// to native
JSBool jsval_to_ushort( JSContext *cx, jsval vp, unsigned short *ret );
JSBool jsval_to_int32( JSContext *cx, jsval vp, int32_t *ret );
JSBool jsval_to_uint32( JSContext *cx, jsval vp, uint32_t *ret );
JSBool jsval_to_uint16( JSContext *cx, jsval vp, uint16_t *ret );
JSBool jsval_to_long( JSContext *cx, jsval vp, long *out);
JSBool jsval_to_ulong( JSContext *cx, jsval vp, unsigned long *out);
JSBool jsval_to_long_long(JSContext *cx, jsval v, long long* ret);
JSBool jsval_to_std_string(JSContext *cx, jsval v, std::string* ret);
JSBool jsval_to_ccpoint(JSContext *cx, jsval v, cocos2d::Point* ret);
JSBool jsval_to_ccrect(JSContext *cx, jsval v, cocos2d::Rect* ret);
JSBool jsval_to_ccsize(JSContext *cx, jsval v, cocos2d::Size* ret);
JSBool jsval_to_cccolor4b(JSContext *cx, jsval v, cocos2d::Color4B* ret);
JSBool jsval_to_cccolor4f(JSContext *cx, jsval v, cocos2d::Color4F* ret);
JSBool jsval_to_cccolor3b(JSContext *cx, jsval v, cocos2d::Color3B* ret);
JSBool jsval_to_ccarray_of_CCPoint(JSContext* cx, jsval v, cocos2d::Point **points, int *numPoints);
JSBool jsval_to_ccarray(JSContext* cx, jsval v, cocos2d::__Array** ret);
JSBool jsval_to_ccdictionary(JSContext* cx, jsval v, cocos2d::__Dictionary** ret);
JSBool jsval_to_ccacceleration(JSContext* cx,jsval v, cocos2d::Acceleration* ret);
JSBool jsvals_variadic_to_ccarray( JSContext *cx, jsval *vp, int argc, cocos2d::__Array** ret);
// forward declaration
js_proxy_t* jsb_get_js_proxy(JSObject* jsObj);
template <class T>
JSBool jsvals_variadic_to_ccvector( JSContext *cx, jsval *vp, int argc, cocos2d::Vector<T>* ret);
JSBool jsvals_variadic_to_ccvaluevector( JSContext *cx, jsval *vp, int argc, cocos2d::ValueVector* ret);
JSBool jsval_to_ccaffinetransform(JSContext* cx, jsval v, cocos2d::AffineTransform* ret);
JSBool jsval_to_FontDefinition( JSContext *cx, jsval vp, cocos2d::FontDefinition* ret );
template <class T>
JSBool jsval_to_ccvector(JSContext* cx, jsval v, cocos2d::Vector<T>* ret);
JSBool jsval_to_ccvalue(JSContext* cx, jsval v, cocos2d::Value* ret);
JSBool jsval_to_ccvaluemap(JSContext* cx, jsval v, cocos2d::ValueMap* ret);
JSBool jsval_to_ccvaluemapintkey(JSContext* cx, jsval v, cocos2d::ValueMapIntKey* ret);
JSBool jsval_to_ccvaluevector(JSContext* cx, jsval v, cocos2d::ValueVector* ret);
JSBool jsval_to_ssize( JSContext *cx, jsval vp, ssize_t* ret);
JSBool jsval_to_std_vector_string( JSContext *cx, jsval vp, std::vector<std::string>* ret);
JSBool jsval_to_std_vector_int( JSContext *cx, jsval vp, std::vector<int>* ret);
template <class T>
JSBool jsval_to_ccmap_string_key(JSContext *cx, jsval v, cocos2d::Map<std::string, T>* ret);
// from native
jsval int32_to_jsval( JSContext *cx, int32_t l);
jsval uint32_to_jsval( JSContext *cx, uint32_t number );
jsval ushort_to_jsval( JSContext *cx, unsigned short number );
jsval long_to_jsval( JSContext *cx, long number );
jsval ulong_to_jsval(JSContext* cx, unsigned long v);
jsval long_long_to_jsval(JSContext* cx, long long v);
jsval std_string_to_jsval(JSContext* cx, const std::string& v);
jsval c_string_to_jsval(JSContext* cx, const char* v, size_t length = -1);
jsval ccpoint_to_jsval(JSContext* cx, const cocos2d::Point& v);
jsval ccrect_to_jsval(JSContext* cx, const cocos2d::Rect& v);
jsval ccsize_to_jsval(JSContext* cx, const cocos2d::Size& v);
jsval cccolor4b_to_jsval(JSContext* cx, const cocos2d::Color4B& v);
jsval cccolor4f_to_jsval(JSContext* cx, const cocos2d::Color4F& v);
jsval cccolor3b_to_jsval(JSContext* cx, const cocos2d::Color3B& v);
jsval ccdictionary_to_jsval(JSContext* cx, cocos2d::__Dictionary *dict);
jsval ccarray_to_jsval(JSContext* cx, cocos2d::__Array *arr);
jsval ccacceleration_to_jsval(JSContext* cx, const cocos2d::Acceleration& v);
jsval ccaffinetransform_to_jsval(JSContext* cx, const cocos2d::AffineTransform& t);
jsval FontDefinition_to_jsval(JSContext* cx, const cocos2d::FontDefinition& t);
JSBool jsval_to_CGPoint( JSContext *cx, jsval vp, cpVect *out );
jsval CGPoint_to_jsval( JSContext *cx, cpVect p );
#define cpVect_to_jsval CGPoint_to_jsval
#define jsval_to_cpVect jsval_to_CGPoint
template<class T>
js_proxy_t *js_get_or_create_proxy(JSContext *cx, T *native_obj);
template <class T>
jsval ccvector_to_jsval(JSContext* cx, const cocos2d::Vector<T>& v);
template <class T>
jsval ccmap_string_key_to_jsval(JSContext* cx, const cocos2d::Map<std::string, T>& v);
jsval ccvalue_to_jsval(JSContext* cx, const cocos2d::Value& v);
jsval ccvaluemap_to_jsval(JSContext* cx, const cocos2d::ValueMap& v);
jsval ccvaluemapintkey_to_jsval(JSContext* cx, const cocos2d::ValueMapIntKey& v);
jsval ccvaluevector_to_jsval(JSContext* cx, const cocos2d::ValueVector& v);
jsval ssize_to_jsval(JSContext *cx, ssize_t v);
```

从函数命名来看，很明显是两类：1. xxx_to_jsval  2.jsval_to_xxx。 需要注意的是，这里面曾经潜藏着许多 bug，内存泄漏，转换错误，甚至一些手误代码等。有一些在引擎进化的过程中逐步修复了。

另外，最重要的是，可能有一些 cxx-generator 不支持的转换类型存在，这种不支持的类型转换，就必须要手写代码来实现，例如：我曾经用 map 做了一个简单的只有一个层次的 JSON 对象，作为 talkingdata 的自定义事件追踪函数的参数。

如果你不手写转换代码，cxx-generator 会在无法转换的地方，插入一个编译器警告，但是对于 XCode 来说，基本没有人看那东西，因为编译状态是放在另一个菜单下面的，这是一个隐晦错误。如果你遇到了莫名其妙的代码错误，其中用到一些特殊的新类型，你要去检查你的自动绑定代码，里面很可能有这么一个警告。修复他！然后就可以正常运行了。

下篇继续，应该会讲到回调函数的问题了。

