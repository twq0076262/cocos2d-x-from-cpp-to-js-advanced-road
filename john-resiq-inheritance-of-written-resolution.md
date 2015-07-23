# [【cocos2d-x 从 c++ 到 js】05：John Resiq 的继承写法解析](http://goldlion.blog.51cto.com/4127613/1127020)

今天，我们来看看 John Resiq 的继承写法 Simple JavaScript Inheritance。之前已经有很多同行分析过了。这个写法 在cocos2d-x for js 中也被使用，并作了少许改动。我尝试着做一些展开描述。
先贴源码：

```
cc.Class = function(){};  
cc.Class.extend = function (prop) { 
    var _super = this.prototype;  
 
    // Instantiate a base class (but only create the instance, 
    // don't run the init constructor) 
    initializing = true; 
    var prototype = new this(); 
    initializing = false; 
    fnTest = /xyz/.test(function(){xyz;}) ? /\b_super\b/ : /.*/; 
 
    // Copy the properties over onto the new prototype 
    for (var name in prop) { 
        // Check if we're overwriting an existing function 
        prototype[name] = typeof prop[name] == "function" && 
            typeof _super[name] == "function" && fnTest.test(prop[name]) ? 
            (function (name, fn) { 
                return function () { 
                    var tmp = this._super; 
 
                    // Add a new ._super() method that is the same method 
                    // but on the super-class 
                    this._super = _super[name]; 
 
                    // The method only need to be bound temporarily, so we 
                    // remove it when we're done executing 
                    var ret = fn.apply(this, arguments); 
                    this._super = tmp; 
 
                    return ret; 
                }; 
            })(name, prop[name]) : 
            prop[name]; 
    } 
 
    // The dummy class constructor 
    function Class() { 
        // All construction is actually done in the init method 
        if (!initializing && this.ctor) 
            this.ctor.apply(this, arguments); 
    } 
 
    // Populate our constructed prototype object 
    Class.prototype = prototype; 
 
    // Enforce the constructor to be what we expect 
    Class.prototype.constructor = Class; 
 
    // And make this class extendable 
    Class.extend = arguments.callee; 
 
    return Class; 
}; 
```

```
cc.Class = function(){};  
```

做了一个全局构造函数 Class，这个不需要什么解释。

```
cc.Class.extend = function (prop) {  
```

prop 是一个对象字面量，这个对象包含了子类所需要的全部成员变量和成员方法。 extend 函数就在内部遍历这个字面量的属性，然后将这些属性绑定到一个“新的构造函数”（也就是子类的构造函数）的原型上。

```
var _super = this.prototype; 
```

注意，js 里面的这个 this 的类型是在调用时指定的，那么这个 this 实际上是父类构造函数对象。比如你写了一个 MyNode 继承自 cc.Node。相应代码是：

```
var MyNode = cc.Node.extend({ 
    var _super = this.prototype;
... 
}); 
```

那么这个 this 就是父类 cc.Node。

```
initializing = true; 
var prototype = new this(); 
initializing = false; 
```

生成父类的对象，用于给子类绑定原型链。但要注意，因为这个时候，什么实参都没有，并不应该给父类对象中的属性进行初始化（构造器参数神马的木有怎么初始化啊喵，这玩意实际是 JS 语言设计上的失误造成的）。所以用一个变量做标记，防止在这个时候进行初始化。相关代码在后面就会看到。

```
fnTest = /xyz/.test(function(){xyz;}) ? /\b_super\b/ : /.*/;  
```

这玩意看起来很乱，这是一个正则对象，右边是一个？表达式，中间添加了一些正则代码。这个对象的作用是，检测子类函数中是否有调用父类的同名方法“_super()”(这种调用父类方式是由 John Resiq 约定的)。但这种检测需要 JS 解释器支持把一个函数转换为字符串的功能，有些解释器是不支持的。所以我们先做一个检测，自己造了一个函数，里面有 xyz，然后用正则的 test 函数在里面搜索 xyz。如果返回 true，表示支持函数转字符串，那么就直接返回/\b_super\b/否则返回/.*/

```
 for (var name in prop) {  
        prototype[name] = typeof prop[name] == "function" &&  
            typeof _super[name] == "function" && fnTest.test(prop[name]) ?  
            (function (name, fn) {  
                return function () {  
                    var tmp = this._super;  
                    this._super = _super[name];   
                    var ret = fn.apply(this, arguments);   
                    this._super = tmp;
                    return ret;  
                };  
            })(name, prop[name]) :  
            prop[name];  
}  
```

现在重头戏来了，在这个地方我们要把传进来的那个字面量 prop 的属性全都绑定到原型上。这地方又他喵的是一个？表达式，JR 实在太喜欢用这玩意了。首先，forin 把属性拿出来。然后，因为我们添加的功能是“实现像 c++ 那样通过子类来调用父类的同名函数”，那么需要检测父类和子类中是否都有这两个同名函数。用的是这段代码：

```
typeof prop[name] == "function" && typeof _super[name] == "function" 
```

然后，我们还要检测，子类函数中是否真的使用了_super 去调用了同名的父类函数。这个时候，之前的正则对象 fnTest 出场。继续之前的话题，如果解释器支持函数转字符串，那么 fnTest.test(prop[name]) 可以正常检测，逻辑正常进行；如果不支持，那么 fnTest.test(prop[name]) 始终返回 true。  
这玩意什么用处，这是一个优化，如果子类函数真的调父类函数了，就做一个特殊的绑定操作（这个操作我们后面马上讲），如果子类函数没有调父类函数，那么就正常绑定。如果没法判断是否调用了（解释器不支持函数转字符串），直接按照调用了那种情况来处理。虽然损失一些性能，但是可以保证不出问题。

```
(function (name, fn) {   
    return function () {   
        var tmp = this._super;   
        this._super = _super[name];    
        var ret = fn.apply(this, arguments);    
        this._super = tmp; 
        return ret;   
    };   
})(name, prop[name]) 
```

继续，上面的就是我们说的那个特殊的绑定操作。在这里，我们做了一个闭包，这里面的 this 是子类对象，跟之前的那个 this 不一样哦。我们利用闭包的特性，保存了一个 _super，这个 _super 被绑定了父类的同名函数_super[name]。然后我们使用 fn.apply(this, arguments)调用子类函数，并保存返回值。因为这是一个闭包，所以根据语法，我们可以在 fn的实现中调用 _super 函数。

```
function Class() {  
   if (!initializing && this.ctor)  
        this.ctor.apply(this, arguments);  
}  
 
Class.prototype = prototype;   
Class.prototype.constructor = Class;   
Class.extend = arguments.callee; 
```

生成一个 Class 构造函数，这个构造函数作为这个大匿名函数的返回值使用。然后里面就是之前说的，初始化保护，防止在绑定原型链的时候初始化。注意后面那个玩意 ctor，在 cocos2d-x for js 中，真正的初始化是二段构造的那个 init，而不是 ctor。在 cocos2d-x for js 的实现中 ctor 里面会调用一个函数 cc.associateWithNative(this, 父类)，这个函数负责后台生成一个 c++ 对象，然后把 c++ 对象和 js 对象绑定到一起。
剩下的是例行公事：绑定子类的原型，修正子类的构造器指向它自己，给子类添加一个同样的 extend 方法。
最后把这个完成的构造函数返回出来。