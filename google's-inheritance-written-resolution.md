# [【cocos2d-x 从 c++ 到 js】06：Google 的继承写法解析](http://goldlion.blog.51cto.com/4127613/1127112)

cocos2d-x for js 中集成了两套继承写法，一套是 JR 的，一套是 google。公司同事使用过 node.js，对 google 的继承方式比较赞同。我就看了一下 Google 的继承代码。
先贴代码：

```
// 1) Google "subclasses" borrowed from closure library 
// This is the recommended way to do it 
// 
cc.inherits = function (childCtor, parentCtor) { 
    /** @constructor */ 
    function tempCtor() {}; 
    tempCtor.prototype = parentCtor.prototype; 
    childCtor.superClass_ = parentCtor.prototype; 
    childCtor.prototype = new tempCtor(); 
    childCtor.prototype.constructor = childCtor; 
 
    // Copy "static" method, but doesn't generate subclasses. 
//  for( var i in parentCtor ) { 
//      childCtor[ i ] = parentCtor[ i ]; 
//  } 
```

cc.inherits 是继承函数，负责链接父类和子类的原型链。非常有趣的是，在这里使用了一个临时构造器，这样就替换了 JR 代码中的 initializing 写法。看起来很舒服。

```
cc.base = function(me, opt_methodName, var_args) {  
    var caller = arguments.callee.caller;  
    if (caller.superClass_) {  
        // This is a constructor. Call the superclass constructor.  
        ret =  caller.superClass_.constructor.apply( me, Array.prototype.slice.call(arguments, 1));  
        return ret;  
    }  
  
    var args = Array.prototype.slice.call(arguments, 2);  
    var foundCaller = false;  
    for (var ctor = me.constructor;  
        ctor; ctor = ctor.superClass_ && ctor.superClass_.constructor) {  
        if (ctor.prototype[opt_methodName] === caller) {  
            foundCaller = true;  
        } else if (foundCaller) {  
            return ctor.prototype[opt_methodName].apply(me, args);  
        }  
    }  
  
    // If we did not find the caller in the prototype chain,  
    // then one of two things happened:  
    // 1) The caller is an instance method.  
    // 2) This method was not called by the right caller.  
    if (me[opt_methodName] === caller) {  
        return me.constructor.prototype[opt_methodName].apply(me, args);  
    } else {  
        throw Error(  
                    'cc.base called from a method of one name ' +  
                    'to a method of a different name');  
    }  
};  
```

cc.base 是在子类函数中调用父类同名函数的方法。要使用这个函数，必须是使用过 cc.inherits 进行过链接原型链的类才行。参数方面，me 需要传入 this，其他根据形参表来定。

```
var caller = arguments.callee.caller; 
```

首先通过，上面的代码获得外层函数的对象。（据说 caller 这个属性已经不再建议使用了，不知道是什么原因）。
然后，如果外层函数是构造函数的话，一定是存在 superClass_ 这个属性的。那么可以用 apply 调用父类的构造器，然后就退出函数执行就可以了。（但是这里为什么会有返回值呢，他喵的构造器返回值不是被运行环境给接管了么？）

```
  var args = Array.prototype.slice.call(arguments, 2);   
    var foundCaller = false;   
    for (var ctor = me.constructor;   
        ctor; ctor = ctor.superClass_ && ctor.superClass_.constructor) {   
        if (ctor.prototype[opt_methodName] === caller) {   
            foundCaller = true;   
        } else if (foundCaller) {   
            return ctor.prototype[opt_methodName].apply(me, args);   
        }   
    }   
```

如果外层函数不是构造函数，那么就是子类的普通函数。后面的代码也很简单，从子类向上往父类上面找，一层一层的遍历构造器，然后再核对同名函数，如果在当前层次找到了对应的函数名，就在下一轮循环中，调用父类的同名函数即可。然后直接返回。

```
if (me[opt_methodName] === caller) {   
        return me.constructor.prototype[opt_methodName].apply(me, args);   
    } else {   
        throw Error(   
                    'cc.base called from a method of one name ' +   
                    'to a method of a different name');   
    }   
```

果要调用的那个函数，既不是构造函数，也不是父类中的同名函数。那么只有一种可能，就是这个函数是子类的一个实例上的函数。直接 apply 调用就好了。
再找不到的话，代码就会抽风了。（throw Error）

综上，google 的代码风格非常流畅，可读性也很高。如果 JR 是很黄很暴力，各种奇技淫巧不计其数。那么 google 的代码就是和风细雨，润物细无声。
就我个人而已，非常喜欢 JR 的接口，但是又喜欢 google 的内部实现。矛盾啊，喵了个咪。
另外，google 的代码可以做到很容易的和其他继承机制兼容，但 JR 的就不行，必须已自己为核心来搞才可以的。这些是由他们的实现机制决定的。
目前来说，cocos2d-x for js 使用 JR 的写法，不知道会不会对将来的扩展造成一些问题呢。

