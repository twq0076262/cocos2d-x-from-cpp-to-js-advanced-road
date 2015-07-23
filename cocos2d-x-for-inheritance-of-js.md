# [【cocos2d-x 从 c++ 到 js】04：cocos2d-x for js 中的继承](http://goldlion.blog.51cto.com/4127613/1125167)

对于面向对象语言来说，继承机制是代码复用的基础，很不幸的是 javascript 作为一个基于原型继承的语言，并没有在本身语言层面上直接作出对类继承的支持。
但是 js 语言拥有很强大的表现力。所以一般是 js 的使用者自行设计一套继承机制，这个机制必须包括几个点，对私有访问权限的模拟，对属性和类属性的不同实现，对方法覆盖的支持，对父类被覆盖方法的访问等。 
cocos2d-x 中，整合了两套继承机制，看《MoonWarriors》例子中的源码SysMenu.js文件 

```
var SysMenu = cc.Layer.extend({ 
    _ship:null, 
 
    ctor:function () { 
        cc.associateWithNative( this, cc.Layer ); 
    }, 
    init:function () { 
        var bRet = false; 
        if (this._super()) { 
            winSize = cc.Director.getInstance().getWinSize(); 
            var sp = cc.Sprite.create(s_loading); 
            sp.setAnchorPoint(cc.p(0,0)); 
            this.addChild(sp, 0, 1); 
 
            var logo = cc.Sprite.create(s_logo); 
            logo.setAnchorPoint(cc.p(0, 0)); 
            logo.setPosition(0, 250); 
            this.addChild(logo, 10, 1); 
 
            var newGameNormal = cc.Sprite.create(s_menu, cc.rect(0, 0, 126, 33)); 
            var newGameSelected = cc.Sprite.create(s_menu, cc.rect(0, 33, 126, 33)); 
            var newGameDisabled = cc.Sprite.create(s_menu, cc.rect(0, 33 * 2, 126, 33)); 
 
            var gameSettingsNormal = cc.Sprite.create(s_menu, cc.rect(126, 0, 126, 33)); 
            var gameSettingsSelected = cc.Sprite.create(s_menu, cc.rect(126, 33, 126, 33)); 
            var gameSettingsDisabled = cc.Sprite.create(s_menu, cc.rect(126, 33 * 2, 126, 33)); 
 
            var aboutNormal = cc.Sprite.create(s_menu, cc.rect(252, 0, 126, 33)); 
            var aboutSelected = cc.Sprite.create(s_menu, cc.rect(252, 33, 126, 33)); 
            var aboutDisabled = cc.Sprite.create(s_menu, cc.rect(252, 33 * 2, 126, 33)); 
 
            var newGame = cc.MenuItemSprite.create(newGameNormal, newGameSelected, newGameDisabled, function () { 
                this.onButtonEffect(); 
                flareEffect(this, this, this.onNewGame); 
            }.bind(this)); 
            var gameSettings = cc.MenuItemSprite.create(gameSettingsNormal, gameSettingsSelected, gameSettingsDisabled, this.onSettings, this); 
            var about = cc.MenuItemSprite.create(aboutNormal, aboutSelected, aboutDisabled, this.onAbout, this); 
 
            var menu = cc.Menu.create(newGame, gameSettings, about); 
            menu.alignItemsVerticallyWithPadding(10); 
            this.addChild(menu, 1, 2); 
            menu.setPosition(winSize.width / 2, winSize.height / 2 - 80); 
            this.schedule(this.update, 0.1); 
 
            var tmp = cc.TextureCache.getInstance().addImage(s_ship01); 
            this._ship = cc.Sprite.createWithTexture(tmp,cc.rect(0, 45, 60, 38)); 
            this.addChild(this._ship, 0, 4); 
            var pos = cc.p(Math.random() * winSize.width, 0); 
            this._ship.setPosition( pos ); 
            this._ship.runAction(cc.MoveBy.create(2, cc.p(Math.random() * winSize.width, pos.y + winSize.height + 100))); 
 
            if (MW.SOUND) { 
                cc.AudioEngine.getInstance().setMusicVolume(0.7); 
                cc.AudioEngine.getInstance().playMusic(s_mainMainMusic, true); 
            } 
 
            bRet = true; 
        } 
        return bRet; 
    }, 
    onNewGame:function (pSender) { 
        var scene = cc.Scene.create(); 
        scene.addChild(GameLayer.create()); 
        scene.addChild(GameControlMenu.create()); 
        cc.Director.getInstance().replaceScene(cc.TransitionFade.create(1.2, scene)); 
    }, 
    onSettings:function (pSender) { 
        this.onButtonEffect(); 
        var scene = cc.Scene.create(); 
        scene.addChild(SettingsLayer.create()); 
        cc.Director.getInstance().replaceScene(cc.TransitionFade.create(1.2, scene)); 
    }, 
    onAbout:function (pSender) { 
        this.onButtonEffect(); 
        var scene = cc.Scene.create(); 
        scene.addChild(AboutLayer.create()); 
        cc.Director.getInstance().replaceScene(cc.TransitionFade.create(1.2, scene)); 
    }, 
    update:function () { 
        if (this._ship.getPosition().y > 480) { 
            var pos = cc.p(Math.random() * winSize.width, 10); 
            this._ship.setPosition( pos ); 
            this._ship.runAction( cc.MoveBy.create( 
                parseInt(5 * Math.random(), 10), 
                cc.p(Math.random() * winSize.width, pos.y + 480))); 
        } 
    }, 
    onButtonEffect:function(){ 
        if (MW.SOUND) { 
            var s = cc.AudioEngine.getInstance().playEffect(s_buttonEffect); 
        } 
    } 
}); 
 
SysMenu.create = function () { 
    var sg = new SysMenu(); 
    if (sg && sg.init()) { 
        return sg; 
    } 
    return null; 
}; 
 
SysMenu.scene = function () { 
    var scene = cc.Scene.create(); 
    var layer = SysMenu.create(); 
    scene.addChild(layer); 
    return scene; 
}; 
```

这个 extend 继承写法由 John Resig 创造，John Resig 是 JS 领域的大神，而且网上有很多粉丝给他编的段子，非常有趣。
例子中使用父类 cc.Layer.extend 方法来启动继承，传入一个对象字面量{}，这个字面量可以包含对象属性和对象方法，最终由 extend 来完成接口绑定，返回一个构造函数赋值给 SysMenu。
对于类方法（也就是通常意义上的静态方法），使用的是 js 最传统的方式，直接给构造函数指定属性即可。
这种编写代码的方式非常简单，而且也很优美。更重要的是，这种写法，非常符合 C++ 或 java 程序员的排版审美。
关于继承的理解。js 里面的原型继承和基于类的继承方式截然不同，内部是在维护一个原型链，链上的节点与节点之间是链接关系（注意：不是赋值，也不是拷贝）。可以先看一下《权威指南》那本书是怎么讲的，不过很遗憾，那本书关于原型继承的图解画的不太好……千万不要搞代数式的替换和死记硬背，那样你很难掌握原型链的本质。
另外，强烈推荐三生石上的系列文章《JavaScript 继承详解》

[JavaScript继承详解](http://www.cnblogs.com/sanshi/archive/2009/07/08/1519036.html)
[JavaScript继承详解（二）](http://www.cnblogs.com/sanshi/archive/2009/07/08/1519251.html)
[JavaScript继承详解（三）](http://www.cnblogs.com/sanshi/archive/2009/07/09/1519890.html)
[JavaScript继承详解（四）](http://www.cnblogs.com/sanshi/archive/2009/07/13/1522647.html)
[JavaScript继承详解（五）](http://www.cnblogs.com/sanshi/archive/2009/07/14/1523523.html)
[JavaScript继承详解（六）](http://www.cnblogs.com/sanshi/archive/2009/07/15/1524263.html)

有时间的话，我会把三生石上的文章配一些详细的原型链描述图，这样就可以很容易的掌握 js 的原型链了。