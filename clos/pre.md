<link href="/css/markdown.css" rel="stylesheet"></link>

And now for something completely different: the inheritance!

>Inheritance? Oh, come on! How much of it yet! Go and find a bunch of
 that shit on the internet!

We introduce singular, simple, profitable multiple inheritance system.
It's built entirely of prototypes (and not a single array lookup), and
it'll make your soul relaxed and your heart warm especially if you've heard
anything of Io or Lua or Self or whatever prototypal language
other than javascript. Enter the whole forest of a Class!

## Mushroom Inheritance

### Fore-Story

We need an `__extend` function:

    function __extend(o,e){
      for(var k in e){
        o[k]=e[k]}
      return o;
    }

a `__clone` function:

    function __clone(o){
      return {__proto__:o}
    }

and pretty like nothing more.

Let's put them together and call the thing `Obj`:

    var Obj={
      extend:function(e){
        return __extend(this,e)},
      clone:function(){
        return __clone(this)},
    }
    
Now we could create all kinds of things just like this:

    var Wheeled=Obj.clone().extend({
      wheels:0,
      go:function(){console.log("tag dag dag da")},
    })

    var Bike=Wheeled.clone().extend({
      owner_name:"nobody",
      go:function(){console.log("hey, "+this.owner_name+", din don!")},
      init:function(owner){this.owner_name=owner;return this}
    })

    var b0=Bike.clone().init("Mike")
    b0.go()

    >> hey, Mike, din don!

To keep instances a bit different from classes let's do this way:

    function __delta(o){
      return delta}
    
    Obj.extend({
      new:function(){
        var d0=__delta(this)
        d0.init && d0.init.apply(d0,arguments)
        return d0}})

So now we can do the last bit of bike pirouette like this:

    var b0=Bike.new("Mike")
    b0.go()

    >> hey, Mike, din don!

That would suffice for any semi-complex task of javascript developer, and
that makes us content with terminology to be used in next article.

But to be even more happy, we'd spend another minute or two here.
What if we embed dictionaries in our nifty objects? Surely we
would have them embedded. And if we want them to be modifiable from leaves-instances,
but yet stay unchanged in class-instances, we could do like this:

    function __extend(o,e){
      return o}
    
    function __delta(o){
      return __clone(o)}

So that's like it. And all mushrooms [following](/clos/).

        

