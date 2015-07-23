# [【cocos2d-x 从 c++ 到 js】14：注册函数](http://goldlion.blog.51cto.com/4127613/1357617)

前面的文章中讲过，在游戏启动时，会调用大量的 addRegisterCallback 函数，向 SpiderMonkey 注册 Cocos2d-x 引擎的函数。

```
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
sc->start();
```

以 register_all_cocos2dx 注册函数为例，跳转到实现代码：

```
void register_all_cocos2dx(JSContext* cx, JSObject* obj) {
    // first, try to get the ns
    JS::RootedValue nsval(cx);
    JSObject *ns;
    JS_GetProperty(cx, obj, "cc", &nsval);
    if (nsval == JSVAL_VOID) {
        ns = JS_NewObject(cx, NULL, NULL, NULL);
        nsval = OBJECT_TO_JSVAL(ns);
        JS_SetProperty(cx, obj, "cc", nsval);
    } else {
        JS_ValueToObject(cx, nsval, &ns);
    }
    obj = ns;
    js_register_cocos2dx_Action(cx, obj);
    js_register_cocos2dx_FiniteTimeAction(cx, obj);
    js_register_cocos2dx_ActionInstant(cx, obj);
    js_register_cocos2dx_Hide(cx, obj);
    js_register_cocos2dx_Node(cx, obj);
    js_register_cocos2dx_Scene(cx, obj);
    js_register_cocos2dx_TransitionScene(cx, obj);
    js_register_cocos2dx_TransitionEaseScene(cx, obj);
    js_register_cocos2dx_TransitionMoveInL(cx, obj);
    js_register_cocos2dx_TransitionMoveInB(cx, obj);
    js_register_cocos2dx_Layer(cx, obj);
    js_register_cocos2dx___LayerRGBA(cx, obj);
    js_register_cocos2dx_AtlasNode(cx, obj);
    js_register_cocos2dx_TileMapAtlas(cx, obj);
    js_register_cocos2dx_TransitionMoveInT(cx, obj);
    js_register_cocos2dx_TransitionMoveInR(cx, obj);
    js_register_cocos2dx_ParticleSystem(cx, obj);
    js_register_cocos2dx_ParticleSystemQuad(cx, obj);
    js_register_cocos2dx_ParticleSnow(cx, obj);
    js_register_cocos2dx_ActionInterval(cx, obj);
    js_register_cocos2dx_ActionCamera(cx, obj);
    js_register_cocos2dx_ProgressFromTo(cx, obj);
    js_register_cocos2dx_MoveBy(cx, obj);
    js_register_cocos2dx_MoveTo(cx, obj);
    js_register_cocos2dx_JumpBy(cx, obj);
    js_register_cocos2dx_ActionEase(cx, obj);
    js_register_cocos2dx_EaseBounce(cx, obj);
    js_register_cocos2dx_EaseBounceIn(cx, obj);
    js_register_cocos2dx_TransitionRotoZoom(cx, obj);
    js_register_cocos2dx_Director(cx, obj);
    js_register_cocos2dx_Texture2D(cx, obj);
    js_register_cocos2dx_EaseElastic(cx, obj);
    js_register_cocos2dx_EaseElasticOut(cx, obj);
    js_register_cocos2dx_EaseBackOut(cx, obj);
    js_register_cocos2dx_TransitionSceneOriented(cx, obj);
    js_register_cocos2dx_TransitionFlipX(cx, obj);
    js_register_cocos2dx_Spawn(cx, obj);
    js_register_cocos2dx_SimpleAudioEngine(cx, obj);
    js_register_cocos2dx_SkewTo(cx, obj);
    js_register_cocos2dx_SkewBy(cx, obj);
    js_register_cocos2dx_TransitionProgress(cx, obj);
    js_register_cocos2dx_TransitionProgressVertical(cx, obj);
    js_register_cocos2dx_TMXTiledMap(cx, obj);
    js_register_cocos2dx_GridAction(cx, obj);
    js_register_cocos2dx_Grid3DAction(cx, obj);
    js_register_cocos2dx_FadeIn(cx, obj);
    js_register_cocos2dx_AnimationCache(cx, obj);
    js_register_cocos2dx_FlipX3D(cx, obj);
    js_register_cocos2dx_FlipY3D(cx, obj);
    js_register_cocos2dx_EaseSineInOut(cx, obj);
    js_register_cocos2dx_TransitionFlipAngular(cx, obj);
    js_register_cocos2dx_EGLViewProtocol(cx, obj);
    js_register_cocos2dx_EGLView(cx, obj);
    js_register_cocos2dx_EaseElasticInOut(cx, obj);
    js_register_cocos2dx_Show(cx, obj);
    js_register_cocos2dx_FadeOut(cx, obj);
    js_register_cocos2dx_CallFunc(cx, obj);
    js_register_cocos2dx_Waves3D(cx, obj);
    js_register_cocos2dx_ParticleFireworks(cx, obj);
    js_register_cocos2dx_MenuItem(cx, obj);
    js_register_cocos2dx_MenuItemSprite(cx, obj);
    js_register_cocos2dx_MenuItemImage(cx, obj);
    js_register_cocos2dx_ParticleFire(cx, obj);
    js_register_cocos2dx_TransitionZoomFlipAngular(cx, obj);
    js_register_cocos2dx_EaseRateAction(cx, obj);
    js_register_cocos2dx_EaseIn(cx, obj);
    js_register_cocos2dx_EaseExponentialInOut(cx, obj);
    js_register_cocos2dx_EaseBackInOut(cx, obj);
    js_register_cocos2dx_EaseExponentialOut(cx, obj);
    js_register_cocos2dx_SpriteBatchNode(cx, obj);
    js_register_cocos2dx_Label(cx, obj);
    js_register_cocos2dx_Application(cx, obj);
    js_register_cocos2dx_DelayTime(cx, obj);
    js_register_cocos2dx_LabelAtlas(cx, obj);
    js_register_cocos2dx_LabelBMFont(cx, obj);
    js_register_cocos2dx_TransitionFadeTR(cx, obj);
    js_register_cocos2dx_TransitionFadeBL(cx, obj);
    js_register_cocos2dx_EaseElasticIn(cx, obj);
    js_register_cocos2dx_ParticleSpiral(cx, obj);
    js_register_cocos2dx_TiledGrid3DAction(cx, obj);
    js_register_cocos2dx_FadeOutTRTiles(cx, obj);
    js_register_cocos2dx_FadeOutUpTiles(cx, obj);
    js_register_cocos2dx_FadeOutDownTiles(cx, obj);
    js_register_cocos2dx_TextureCache(cx, obj);
    js_register_cocos2dx_ActionTween(cx, obj);
    js_register_cocos2dx_TransitionFadeDown(cx, obj);
    js_register_cocos2dx_ParticleSun(cx, obj);
    js_register_cocos2dx_TransitionProgressHorizontal(cx, obj);
    js_register_cocos2dx_TMXObjectGroup(cx, obj);
    js_register_cocos2dx_TMXLayer(cx, obj);
    js_register_cocos2dx_FlipX(cx, obj);
    js_register_cocos2dx_FlipY(cx, obj);
    js_register_cocos2dx_TransitionSplitCols(cx, obj);
    js_register_cocos2dx_Timer(cx, obj);
    js_register_cocos2dx_FadeTo(cx, obj);
    js_register_cocos2dx_Repeat(cx, obj);
    js_register_cocos2dx_Place(cx, obj);
    js_register_cocos2dx_GLProgram(cx, obj);
    js_register_cocos2dx_EaseBounceOut(cx, obj);
    js_register_cocos2dx_RenderTexture(cx, obj);
    js_register_cocos2dx_TintBy(cx, obj);
    js_register_cocos2dx_TransitionShrinkGrow(cx, obj);
    js_register_cocos2dx_Sprite(cx, obj);
    js_register_cocos2dx_LabelTTF(cx, obj);
    js_register_cocos2dx_ClippingNode(cx, obj);
    js_register_cocos2dx_ParticleFlower(cx, obj);
    js_register_cocos2dx_ParticleSmoke(cx, obj);
    js_register_cocos2dx_LayerMultiplex(cx, obj);
    js_register_cocos2dx_Blink(cx, obj);
    js_register_cocos2dx_ShaderCache(cx, obj);
    js_register_cocos2dx_JumpTo(cx, obj);
    js_register_cocos2dx_ParticleExplosion(cx, obj);
    js_register_cocos2dx_TransitionJumpZoom(cx, obj);
    js_register_cocos2dx_Touch(cx, obj);
    js_register_cocos2dx_AnimationFrame(cx, obj);
    js_register_cocos2dx_NodeGrid(cx, obj);
    js_register_cocos2dx_TMXLayerInfo(cx, obj);
    js_register_cocos2dx_TMXTilesetInfo(cx, obj);
    js_register_cocos2dx_GridBase(cx, obj);
    js_register_cocos2dx_TiledGrid3D(cx, obj);
    js_register_cocos2dx_ParticleGalaxy(cx, obj);
    js_register_cocos2dx_Twirl(cx, obj);
    js_register_cocos2dx_MenuItemLabel(cx, obj);
    js_register_cocos2dx_LayerColor(cx, obj);
    js_register_cocos2dx_FadeOutBLTiles(cx, obj);
    js_register_cocos2dx_LayerGradient(cx, obj);
    js_register_cocos2dx_TargetedAction(cx, obj);
    js_register_cocos2dx_RepeatForever(cx, obj);
    js_register_cocos2dx_CardinalSplineTo(cx, obj);
    js_register_cocos2dx_CardinalSplineBy(cx, obj);
    js_register_cocos2dx_TransitionFlipY(cx, obj);
    js_register_cocos2dx_TurnOffTiles(cx, obj);
    js_register_cocos2dx_TintTo(cx, obj);
    js_register_cocos2dx_CatmullRomTo(cx, obj);
    js_register_cocos2dx_ToggleVisibility(cx, obj);
    js_register_cocos2dx_DrawNode(cx, obj);
    js_register_cocos2dx_TransitionTurnOffTiles(cx, obj);
    js_register_cocos2dx_RotateTo(cx, obj);
    js_register_cocos2dx_TransitionSplitRows(cx, obj);
    js_register_cocos2dx_TransitionProgressRadialCCW(cx, obj);
    js_register_cocos2dx_ScaleTo(cx, obj);
    js_register_cocos2dx_TransitionPageTurn(cx, obj);
    js_register_cocos2dx_BezierBy(cx, obj);
    js_register_cocos2dx_BezierTo(cx, obj);
    js_register_cocos2dx_Menu(cx, obj);
    js_register_cocos2dx_SpriteFrame(cx, obj);
    js_register_cocos2dx_ActionManager(cx, obj);
    js_register_cocos2dx_TransitionFade(cx, obj);
    js_register_cocos2dx_TransitionZoomFlipX(cx, obj);
    js_register_cocos2dx_SpriteFrameCache(cx, obj);
    js_register_cocos2dx_TransitionCrossFade(cx, obj);
    js_register_cocos2dx_Ripple3D(cx, obj);
    js_register_cocos2dx_TransitionSlideInL(cx, obj);
    js_register_cocos2dx_TransitionSlideInT(cx, obj);
    js_register_cocos2dx_StopGrid(cx, obj);
    js_register_cocos2dx_ShakyTiles3D(cx, obj);
    js_register_cocos2dx_PageTurn3D(cx, obj);
    js_register_cocos2dx_Grid3D(cx, obj);
    js_register_cocos2dx_TransitionProgressInOut(cx, obj);
    js_register_cocos2dx_EaseBackIn(cx, obj);
    js_register_cocos2dx_SplitRows(cx, obj);
    js_register_cocos2dx_Follow(cx, obj);
    js_register_cocos2dx_Animate(cx, obj);
    js_register_cocos2dx_ShuffleTiles(cx, obj);
    js_register_cocos2dx_ProgressTimer(cx, obj);
    js_register_cocos2dx_ParticleMeteor(cx, obj);
    js_register_cocos2dx_EaseInOut(cx, obj);
    js_register_cocos2dx_TransitionZoomFlipY(cx, obj);
    js_register_cocos2dx_ScaleBy(cx, obj);
    js_register_cocos2dx_Lens3D(cx, obj);
    js_register_cocos2dx_Animation(cx, obj);
    js_register_cocos2dx_TMXMapInfo(cx, obj);
    js_register_cocos2dx_EaseExponentialIn(cx, obj);
    js_register_cocos2dx_ReuseGrid(cx, obj);
    js_register_cocos2dx_MenuItemAtlasFont(cx, obj);
    js_register_cocos2dx_Liquid(cx, obj);
    js_register_cocos2dx_OrbitCamera(cx, obj);
    js_register_cocos2dx_ParticleBatchNode(cx, obj);
    js_register_cocos2dx_Component(cx, obj);
    js_register_cocos2dx_TextFieldTTF(cx, obj);
    js_register_cocos2dx_ParticleRain(cx, obj);
    js_register_cocos2dx_Waves(cx, obj);
    js_register_cocos2dx_EaseOut(cx, obj);
    js_register_cocos2dx_MenuItemFont(cx, obj);
    js_register_cocos2dx_TransitionFadeUp(cx, obj);
    js_register_cocos2dx_EaseSineOut(cx, obj);
    js_register_cocos2dx_JumpTiles3D(cx, obj);
    js_register_cocos2dx_MenuItemToggle(cx, obj);
    js_register_cocos2dx_RemoveSelf(cx, obj);
    js_register_cocos2dx_SplitCols(cx, obj);
    js_register_cocos2dx_MotionStreak(cx, obj);
    js_register_cocos2dx_RotateBy(cx, obj);
    js_register_cocos2dx_FileUtils(cx, obj);
    js_register_cocos2dx_ProgressTo(cx, obj);
    js_register_cocos2dx_TransitionProgressOutIn(cx, obj);
    js_register_cocos2dx_CatmullRomBy(cx, obj);
    js_register_cocos2dx_Sequence(cx, obj);
    js_register_cocos2dx_Shaky3D(cx, obj);
    js_register_cocos2dx_TransitionProgressRadialCW(cx, obj);
    js_register_cocos2dx_EaseBounceInOut(cx, obj);
    js_register_cocos2dx_TransitionSlideInR(cx, obj);
    js_register_cocos2dx___NodeRGBA(cx, obj);
    js_register_cocos2dx_ParallaxNode(cx, obj);
    js_register_cocos2dx_Scheduler(cx, obj);
    js_register_cocos2dx_EaseSineIn(cx, obj);
    js_register_cocos2dx_WavesTiles3D(cx, obj);
    js_register_cocos2dx_TransitionSlideInB(cx, obj);
    js_register_cocos2dx_Speed(cx, obj);
    js_register_cocos2dx_ShatteredTiles3D(cx, obj);
}
```

首先看到的是从根对象中获取一个“cc”属性（如果获取不到，就新建一个），因为 JS 中没有名字空间的概念，所以我们使用一个 cc 对象来表示类似的功能。所有的类型和函数都是这个 cc 对象下面的属性。在 Cocos2d-x 3.0 中，C++ 层面，类名去掉了 CC 的前缀，和 js 保持一致。

然后就是一大堆子函数，每个函数都负责注册一个对应的类。打开 js_register_cocos2dx_Sprite，这个函数负责注册 Sprite 类。

打开 js_register_cocos2dx_Sprite 的实现代码

```
void js_register_cocos2dx_Sprite(JSContext *cx, JSObject *global) {
    jsb_cocos2d_Sprite_class = (JSClass *)calloc(1, sizeof(JSClass));
    jsb_cocos2d_Sprite_class->name = "Sprite";
    jsb_cocos2d_Sprite_class->addProperty = JS_PropertyStub;
    jsb_cocos2d_Sprite_class->delProperty = JS_DeletePropertyStub;
    jsb_cocos2d_Sprite_class->getProperty = JS_PropertyStub;
    jsb_cocos2d_Sprite_class->setProperty = JS_StrictPropertyStub;
    jsb_cocos2d_Sprite_class->enumerate = JS_EnumerateStub;
    jsb_cocos2d_Sprite_class->resolve = JS_ResolveStub;
    jsb_cocos2d_Sprite_class->convert = JS_ConvertStub;
    jsb_cocos2d_Sprite_class->finalize = js_cocos2d_Sprite_finalize;
    jsb_cocos2d_Sprite_class->flags = JSCLASS_HAS_RESERVED_SLOTS(2);
    static JSPropertySpec properties[] = {
        {0, 0, 0, JSOP_NULLWRAPPER, JSOP_NULLWRAPPER}
    };
    static JSFunctionSpec funcs[] = {
        JS_FN("setSpriteFrame", js_cocos2dx_Sprite_setSpriteFrame, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("setTexture", js_cocos2dx_Sprite_setTexture, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("getTexture", js_cocos2dx_Sprite_getTexture, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("setFlippedY", js_cocos2dx_Sprite_setFlippedY, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("setFlippedX", js_cocos2dx_Sprite_setFlippedX, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("getBatchNode", js_cocos2dx_Sprite_getBatchNode, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("getOffsetPosition", js_cocos2dx_Sprite_getOffsetPosition, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("removeAllChildrenWithCleanup", js_cocos2dx_Sprite_removeAllChildrenWithCleanup, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("updateQuadVertices", js_cocos2dx_Sprite_updateQuadVertices, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("updateTransform", js_cocos2dx_Sprite_updateTransform, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("setTextureRect", js_cocos2dx_Sprite_setTextureRect, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("isFrameDisplayed", js_cocos2dx_Sprite_isFrameDisplayed, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("getAtlasIndex", js_cocos2dx_Sprite_getAtlasIndex, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("setBatchNode", js_cocos2dx_Sprite_setBatchNode, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("setDisplayFrameWithAnimationName", js_cocos2dx_Sprite_setDisplayFrameWithAnimationName, 2, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("setTextureAtlas", js_cocos2dx_Sprite_setTextureAtlas, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("getSpriteFrame", js_cocos2dx_Sprite_getSpriteFrame, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("isDirty", js_cocos2dx_Sprite_isDirty, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("setAtlasIndex", js_cocos2dx_Sprite_setAtlasIndex, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("setDirty", js_cocos2dx_Sprite_setDirty, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("isTextureRectRotated", js_cocos2dx_Sprite_isTextureRectRotated, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("getTextureRect", js_cocos2dx_Sprite_getTextureRect, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("getTextureAtlas", js_cocos2dx_Sprite_getTextureAtlas, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("isFlippedX", js_cocos2dx_Sprite_isFlippedX, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("isFlippedY", js_cocos2dx_Sprite_isFlippedY, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("setVertexRect", js_cocos2dx_Sprite_setVertexRect, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("ctor", js_cocos2d_Sprite_ctor, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FS_END
    };
    static JSFunctionSpec st_funcs[] = {
        JS_FN("create", js_cocos2dx_Sprite_create, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("createWithTexture", js_cocos2dx_Sprite_createWithTexture, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("createWithSpriteFrameName", js_cocos2dx_Sprite_createWithSpriteFrameName, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FN("createWithSpriteFrame", js_cocos2dx_Sprite_createWithSpriteFrame, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
        JS_FS_END
    };
    jsb_cocos2d_Sprite_prototype = JS_InitClass(
        cx, global,
        jsb_cocos2d_Node_prototype,
        jsb_cocos2d_Sprite_class,
        dummy_constructor<cocos2d::Sprite>, 0, // no constructor
        properties,
        funcs,
        NULL, // no static properties
        st_funcs);
    // make the class enumerable in the registered namespace
    JSBool found;
    JS_SetPropertyAttributes(cx, global, "Sprite", JSPROP_ENUMERATE | JSPROP_READONLY, &found);
    // add the proto and JSClass to the type->js info hash table
    TypeTest<cocos2d::Sprite> t;
    js_type_class_t *p;
    std::string typeName = t.s_name();
    if (_js_global_type_map.find(typeName) == _js_global_type_map.end())
    {
        p = (js_type_class_t *)malloc(sizeof(js_type_class_t));
        p->jsclass = jsb_cocos2d_Sprite_class;
        p->proto = jsb_cocos2d_Sprite_prototype;
        p->parentProto = jsb_cocos2d_Node_prototype;
        _js_global_type_map.insert(std::make_pair(typeName, p));
    }
}
```

看起来比较长，其实很简单，我们一段一段分析。

```
jsb_cocos2d_Sprite_class = (JSClass *)calloc(1, sizeof(JSClass));
jsb_cocos2d_Sprite_class->name = "Sprite";
jsb_cocos2d_Sprite_class->addProperty = JS_PropertyStub;
jsb_cocos2d_Sprite_class->delProperty = JS_DeletePropertyStub;
jsb_cocos2d_Sprite_class->getProperty = JS_PropertyStub;
jsb_cocos2d_Sprite_class->setProperty = JS_StrictPropertyStub;
jsb_cocos2d_Sprite_class->enumerate = JS_EnumerateStub;
jsb_cocos2d_Sprite_class->resolve = JS_ResolveStub;
jsb_cocos2d_Sprite_class->convert = JS_ConvertStub;
jsb_cocos2d_Sprite_class->finalize = js_cocos2d_Sprite_finalize;
```

首先，我们构造了一个 JSClass 对象，这个对象保存了一部分 Sprite 类的相关信息（注意，只是一部分而已）。其中包括类名，还有大量函数的占位符 JS_XXXStub，这些函数是在一定情况下被调用的，如：添加删除属性，查看修改属性等等。这块其实不用特别关注，因为使用的都是 SpiderMonkey 自带的缺省实现。Cocos2d-x 引擎只是在最后把 finalize 函数替换成自己的函数了。最后那个参数表示这个类，有几个 Reserved Slots 槽，这东西我们在之前讲回调函数的时候见过。

```
static JSPropertySpec properties[] = {
    {0, 0, 0, JSOP_NULLWRAPPER, JSOP_NULLWRAPPER}
};
static JSFunctionSpec funcs[] = {
    JS_FN("setSpriteFrame", js_cocos2dx_Sprite_setSpriteFrame, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("setTexture", js_cocos2dx_Sprite_setTexture, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("getTexture", js_cocos2dx_Sprite_getTexture, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("setFlippedY", js_cocos2dx_Sprite_setFlippedY, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("setFlippedX", js_cocos2dx_Sprite_setFlippedX, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("getBatchNode", js_cocos2dx_Sprite_getBatchNode, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("getOffsetPosition", js_cocos2dx_Sprite_getOffsetPosition, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("removeAllChildrenWithCleanup", js_cocos2dx_Sprite_removeAllChildrenWithCleanup, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("updateQuadVertices", js_cocos2dx_Sprite_updateQuadVertices, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("updateTransform", js_cocos2dx_Sprite_updateTransform, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("setTextureRect", js_cocos2dx_Sprite_setTextureRect, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("isFrameDisplayed", js_cocos2dx_Sprite_isFrameDisplayed, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("getAtlasIndex", js_cocos2dx_Sprite_getAtlasIndex, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("setBatchNode", js_cocos2dx_Sprite_setBatchNode, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("setDisplayFrameWithAnimationName", js_cocos2dx_Sprite_setDisplayFrameWithAnimationName, 2, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("setTextureAtlas", js_cocos2dx_Sprite_setTextureAtlas, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("getSpriteFrame", js_cocos2dx_Sprite_getSpriteFrame, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("isDirty", js_cocos2dx_Sprite_isDirty, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("setAtlasIndex", js_cocos2dx_Sprite_setAtlasIndex, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("setDirty", js_cocos2dx_Sprite_setDirty, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("isTextureRectRotated", js_cocos2dx_Sprite_isTextureRectRotated, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("getTextureRect", js_cocos2dx_Sprite_getTextureRect, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("getTextureAtlas", js_cocos2dx_Sprite_getTextureAtlas, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("isFlippedX", js_cocos2dx_Sprite_isFlippedX, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("isFlippedY", js_cocos2dx_Sprite_isFlippedY, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("setVertexRect", js_cocos2dx_Sprite_setVertexRect, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("ctor", js_cocos2d_Sprite_ctor, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FS_END
};
static JSFunctionSpec st_funcs[] = {
    JS_FN("create", js_cocos2dx_Sprite_create, 0, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("createWithTexture", js_cocos2dx_Sprite_createWithTexture, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("createWithSpriteFrameName", js_cocos2dx_Sprite_createWithSpriteFrameName, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FN("createWithSpriteFrame", js_cocos2dx_Sprite_createWithSpriteFrame, 1, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_FS_END
};
```

然后是填写大量的参数，包括属性，静态属性，函数，静态函数。注意，因为 Cocos2d-x 的类完全使用 Setter 和 Getter，所以一般是没有属性和静态属性的。比较重要的是静态函数和普通函数。我们看一下宏函数 JS_FN。他的第一个参数是函数名，这个和 C++ 层的函数命名是一致的，第二个参数在 SpiderMonkey 调用 JS 层对应的 C++ 函数时的回调函数，这个函数我们之前的文章中分析过。第三个参数是函数调用时的参数个数。最后一个参数，是一些访问特性，JSPROP_PERMANENT 表示不可删除，JSPROP_ENUMERATE 表示在枚举时可见（JS 的 for 遍历）。

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
// make the class enumerable in the registered namespace
JSBool found;
JS_SetPropertyAttributes(cx, global, "Sprite", JSPROP_ENUMERATE | JSPROP_READONLY, &found);
```

因为 JS 使用的是原型继承，那么我们需要构造一个原型，需要的参数也很多，都是我们上面配置好的，上下文，父对象，原型，JSClass 对象，各种属性和函数。然后，会自动把这个原型设置为 global（就是之前的 cc 对象）的一个属性。最后，设置好访问特性。

```
TypeTest<cocos2d::Sprite> t;
    js_type_class_t *p;
    std::string typeName = t.s_name();
    if (_js_global_type_map.find(typeName) == _js_global_type_map.end())
    {
        p = (js_type_class_t *)malloc(sizeof(js_type_class_t));
        p->jsclass = jsb_cocos2d_Sprite_class;
        p->proto = jsb_cocos2d_Sprite_prototype;
        p->parentProto = jsb_cocos2d_Node_prototype;
        _js_global_type_map.insert(std::make_pair(typeName, p));
    }
```

最后这段代码是 Cocos2d-x 引擎自己做的一个设计，把类型信息存到一个 map 里，这个设计以后会经常见到。可以用来查询，另外在 JS 虚拟机清空时，也用来遍历删除对应的类型信息。做法是先放到一个 map 里，然后 cleanup 时遍历这个 map 即可。就不赘述了。

