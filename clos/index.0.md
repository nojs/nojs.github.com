<link rel="stylesheet" href="/css/markdown.css"></link>

Was your first object-oriented experience related to C++/Java/Ruby? Then
you should go [left](#left). If you were introduced to OO by
Smalltalk or Self or Lua or Perl or Python go [right](#right). If you are into the
Art of Metaobject Protocol thing, just go [straight ahead](#straight-ahead).

### Left # {#left}

C++ style multiple inheritance makes my brain baked. Well, it's
straightforward in it's own way, call it security or encapsulation.

### Right # {#right}

So, we're into CLOS-style inheritance, with tricky but powerful
interoperability model between classes in inheritance graph. In fact, all
you have is one simple rule how messages propagate along graph edges
by default.

Consider some inheritance graph G. For each class in G linearization
of all it's bases is made, defining method resolution order to be used
in message propagation. And now going back to javascript. We have fine
model of composing classes' methods. We just could use prototype chain
to resolve methods. To do that, we have to order ancestor classes
properly in prototype chain for each class. As javascript prototype
system doesn't allow us to have multiple references to prototypes, we
could just build separate prototype chain of copies for each class. In
this way we don't have joy of runtime sharing of prototype objects,
but we can delay prototype chain creation until time the class
instance is created, so, in fact, this drawback affects only already
created instances (they don't have access to features added to class
after instance instantiation) -- and all instances created after
new feature introduction have access to introduced feature in any of
it's base classes.

### Straight Ahead # {#straight-ahead}

So further we describe implementation attempt of such a system.

As we use pure javascript hashes as a basic building block in our
construction. To distinguish them from arrays, or host objects, we
call them dictionaries or dicts for brevity.

In first step we create simple and well-defined OO-like inheritance
system with single inheritance (1). Then we aim at creating such a
system with multiple inheritance (2), that upon instantiation behaves
like system (1) with regards to practical operations, but have it's
prototype chain built conforming to conditions (0).

## Basic js-style OO

1. `__clone()` function:
2. `__extend()` function:
3. `__delta()` function:

We pack them together for day-to-day use and call the thing `Obj`:

    var Obj={
      clone:function(){
        return __clone(this)},
      extend:function(e){
        return __extend(this,e)},
      new:function(){
        var d0=__delta(this)
        d0.init && d0.init.apply(d0,arguments)
        return d0}}


To be able access super-class methods, we allow passing continuation
to `.extend()` function that accepts one `supr` parameter referring to
definition of directly preceding class:

     function __extend1(c,e){
       if((typeof e)==="function"){
         e=e(c.__proto__)}
       return __extend(c,e)}

     Obj.extend=function(e){
       return __extend1(this,e)}

Also we use convenience function for easy access of superclass' methods:

     function _next(self,method,args){
       if((typeof method)==="function"){
         return method.apply(self,args)}}


Thus, we could create and use 'classes' as simple as this:

    var Wheeled=Obj.clone().extend({
      wheels:0,
      init:function(o){
        o.wheels && (this.wheels=o.wheels)},
      go:function(){
        var ss=[]
        for(var i=0;i<this.wheels;i++){
          ss.push(i%2?"tag":"dag")}
        ss.push("da")
        console.log(ss.join(" "))}})

    var Bike=Wheeled.clone().extend(function(supr){return {
      owner:"Unknown",
      init:function(o){
        o.owner && (this.owner=o.owner)},
      go:function(){
        _next(this,supr.go,arguments)
        console.log("din don, here's "+owner+" going!")}}})

    var b0=Bike.new({wheels:2,owner:"Mike"})
    b0.go()

    >> tag dag da
    >> din don, here's Mike going!

Let's emphasize the intended semantics of `__clone()`, `__extend()` and
`__delta()` functions. First, `__clone()` is just like 'clone' message in Io: it returns new empty
dict with [proto] set to refer to original cloned object. Then, `__extend()` intended to
act on newly created clone (call it C). It overwrites any properties
found directly in C with it's arguments properties, but doesn't affect
any of C's prototypes. And lastly, `__delta()` is used at time of instance
creation: basically it creates 'deep clone' of passed object, so any
property set on embedded dictionaries doesn't affect prototypes' embedded
dictionaries. Method's name is inspired by the fact that we can obtain
clone-obj and cloned-obj difference by inspecting own properties of clone's
embedded dictionaries, so newly created dict is like delta between the two.

Thus, idiomatic way of creating a class
inherited from Base would be:

    var Derived=Base.clone().extend(derived_definition)
    
and creating an instance of Derived:

    var d0=Derived.new(options)

## Mycelium aspects and water transportation

What imposes accidental difficulties in introduced scheme? The thing
that you have to carefully plan for inheritance, and you can't drop in
a class with it's own chain in the middle of other class chain and
expect it to work correctly, or rather you don't
expect anything because you cannot link these two chains together in
any sensible way. So let's try to overcome it.

So in fact, what is a class? Conceptually it's it's properties definition,
name and it's parents (in specified order). Practical aspects of
what class is are the way to instantiate it, the way to pass method
calls between inheritance-adjacent classes, and probably some other
practical issues. Let's for now address conceptual issues and a bit
later return to practical ones. 

    var Aspect=Obj.clone().extend({
      def:function(name,pp,body){},
      extend:function(body){},
      new:function(){}})



<!--  LocalWords:  linearization javascript runtime Lua CLOS dicts OO js
 -->
<!--  LocalWords:  Mycelium
 -->
