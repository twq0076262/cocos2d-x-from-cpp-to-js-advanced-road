# [【cocos2d-x 从 c++ 到 js】03：hybrid 开发模式](http://goldlion.blog.51cto.com/4127613/1120219)

因为苹果是不允许 app 下载可执行代码的，所以用动态链接库构建插件式引擎并通过网络下载在 iOS 上是无法实现的。但有一种方式是，在引擎内部集成一个脚本解释器，然后把脚本作为资源来下载（脚本是加密的），如此规避苹果的审核条款。这个方式就叫 Hybrid。但这么做没法做到不露痕迹，深层原因应该是，Hybrid 牵扯利益太大，苹果也算睁一只眼闭一只眼。
在 cocos2d 引擎的众多分支中，cocos2d-x 的开发是以 C++ 为核心的。而 cocos2d-x 引擎，就是通过 hybrid 方式来执行 js 代码的。为了执行 js 代码，引擎本身需要一个脚本解释器，引擎集成的脚本解释器就叫：spidermonkey。
spidermonkey 是一个历史悠久的基于 c/c++ 编写的 js 脚本解释器，由 Mozilla 提供，非常有名的 firefox 和 thunderbird 都在用。 cocos2d-x 集成 spidermonkey 的开源协议是 MPL2.0，没有什么限制，你可以放心使用它。
在 AppDelegate::applicationDidFinishLaunching() 函数中，我们可以找到启动脚本引擎的代码：

```
ScriptingCore* sc = ScriptingCore::getInstance(); 
sc->addRegisterCallback(register_all_cocos2dx); 
sc->addRegisterCallback(register_cocos2dx_js_extensions); 
sc->addRegisterCallback(register_CCBuilderReader); 
sc->addRegisterCallback(jsb_register_chipmunk); 
sc->start(); 
 
CCScriptEngineProtocol *pEngine = ScriptingCore::getInstance(); 
CCScriptEngineManager::sharedManager()->setScriptEngine(pEngine); 
ScriptingCore::getInstance()->runScript("MoonWarriors-jsb.js"); 
```

ScriptCore 是脚本的核心，他就是我们说的那个 JS 解释器。cocos2d-x 把 spidermonkey 的解释器封装了一下，以提供对 cocos2d-x 引擎的相关支持，并简化相应的调用接口。  
addRegisterCallback 接口用于添加注册函数，注册函数用于在引擎执行时，绑定相应的代码（从JS往C++的映射代码）。每一个注册函数，对应一个库。现在 cocos2d-x 提供了四个库支持，分别是 cocos2d-x 核心库，cocos2d-x 扩展库，cocosbuilder 支持库，clipmunk 物理引擎库。将来你可以在这里添加注册自己实现的 JS 绑定库，来直接扩展这个　JS　引擎。

##start启动脚本引擎。

CCScriptEngineManager::sharedManager()->setScriptEngine 这句是将脚本引擎绑定到引擎管理器上，引擎管理器提供对脚本引擎的一个全局访问点，并且也负责对脚本引擎的卸载。
最后就是运行游戏的主脚本了。

```
ScriptingCore::getInstance()->runScript("MoonWarriors-jsb.js"); 
```