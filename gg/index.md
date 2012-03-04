<link rel="stylesheet" href="/css/markdown.css"></link>


#### Simple Interactive Grammar Generator

    var lx=gg.lexer()
    var E0=gg.expr({
      infix:[
        [["*"],{prec:10,builder:function(e1,op,e2){return ["Op","*",e1,e2]}}]
        [["/"],{prec:10,builder:function(e1,op,e2){return ["Op","/",e1,e2]}}]
        [["+"],{prec:20,builder:function(e1,op,e2){return ["Op","+",e1,e2]}}]
        [["-"],{prec:20,builder:function(e1,op,e2){return ["Op","+",e1,e2]}}]],
      primary:gg.id})
    
    var ls=lx.extract("a+b*c+d/e+f*k/s")
    var e0=E0.parse(ls)
    __eql(e0,
      [Op +
        [Id a]
        [Op +
          [Op * [Id b] [Id c]]
          [Op +
            [Op / [Id d] [Id e]] 
            [Op * [Id f] [Op / [Id k] [Id s]]]]]])

    
      
