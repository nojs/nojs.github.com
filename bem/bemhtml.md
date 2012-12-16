<link rel="stylesheet" href="/css/markdown.css"></link>

## Супер-шаблонизатор!

Отправляюсь вместе с вами в путешествие. Сейчас, когда я пишу эти
строки, bemhtml кажется мне далеким и неизведанным островом, покрытым
таинственными и опасными джунглями.

### Шаблон по-умолчанию

#### Инициализация

Создадим контекст шаблона:

    function BEMContext(context, apply_) {
      this.ctx = context;
      this.apply = apply_;
      this._buf = [];
      this._ = this;

      this._start = true;
      this._mode = '';
      this._listLength = 0;
      this._notNewList = false;
      this.position = 0;
      this.block = undefined;
      this.elem = undefined;
      this.mods = undefined;
      this.elemMods = undefined;
    };


Первым (на старте) обрабатывается шаблон с !this._mode: сначала
isSimple(this.ctx), потом !this.ctx, потом this._.isArray(this.ctx) и потом
режим по-умолчанию true.

Начнем со случая, когда текущий контекст -- простое значение, а
именно, согласно isSimple, когда ctx -- строка, число или boolean.

    BEMContext.prototype.isSimple = function isSimple(obj) {
      var t = typeof obj;
      return t === 'string' || t === 'number' || t === 'boolean';
    };

контекст установлен, но не `boolean` -- положим его в буфер:

    this._.isSimple(this.ctx): {
      this._listLength--;
      var ctx = this.ctx;
      (ctx && ctx !== true || ctx === 0) && this._buf.push(ctx);
    }

`!this.ctx` не сделает ничего, а `this._.isArray(ctx)` разбирает
массив:

    this._.isArray(this.ctx): {
      var v = this.ctx,
      l = v.length,
      i = 0,
      prevPos = this.position,
      prevNotNewList = this._notNewList;
Почему-то:

      if(prevNotNewList) {
        this._listLength += l - 1;
      } else {
        this.position = 0;
        this._listLength = l;
      }

      this._notNewList = true;
И итерация по элементам массива! (В том числе если сюда передан
ctx.content) 

      while(i < l)
        apply(this.ctx = v[i++]);
восстановим `.position`, если не `prevNotNewList`
(если prevNewList, другими словами, т.е. когда в ситуации массив в
массиве мы закончили обрабатывать вложенный массив):

      prevNotNewList || (this.position = prevPos);
    }
Теперь случай по умолчанию:

    true: {
      var vBlock = this.ctx.block, //XXX если когда-либо нужна проверка [1.0] то
                                   //здесь this.ctx будет undefined
      vElem = this.ctx.elem,
Для чего нужны vThing? (Ответ: v используется как сокращение this.ctx)

      block = this._currBlock || this.block;
      this.ctx || (this.ctx = {}); //[1.0]
Создадим контекст!

      local(
        this._mode = 'default',
Это первая мода

        this._links = this.ctx.links || this._links,
Возьмем `links` из контекста. Если они там есть.
        
        this.block = vBlock || (vElem ? block : undefined),
        //vBlock || if(vElem) {block} else {undefined}
        
        this._currBlock = vBlock || vElem ? undefined : block,
Наверное, это блок для обработки элемента:
Если обрабатываем элемент, это или текущий блок с
верхнего уровня, или блок текущего уровня. (Да,
но кто устанавливает this.block?)
        
        this.elem = this.ctx.elem, //ok
        this.mods = (vBlock ? this.ctx.mods : this.mods) || {},
Как и почему определяем здесь модификаторы?

        this.elemMods = this.ctx.elemMods || {}
Модификаторы элемента!

      ) {
Что значат эти {} после local? Видимо, это его scope?

        (this.block || this.elem) ?
          (this.position = (this.position || 0) + 1) :
          this._listLength--;
Туннелируем в моду default!

        apply();
      }
    } 

#### Мода `default`

    default: {
      var _this = this,
      BEM_ = _this.BEM,
      v = this.ctx,
      buf = this._buf,
      tag;
Мода `tag` возвратит тэг из пользовательского шаблона или `undefined`,
и тогда установим ее в `div`:

      tag = apply(this._mode = 'tag');
      typeof tag === 'undefined' && (tag = v.tag);
      typeof tag === 'undefined' && (tag = 'div');

      if(tag) {
I wonder what if not?

        var jsParams, js;
        if(this.block && v.js !== false) {        //ok: we go for js mode
                                               //  if it's in ctx
          js = apply(this._mode = 'js');          //ok
          js = js?                             //ok if(js)
                 this._.extend(v.js,              //ok extend ctx.js with js
                            js === true?
                              {} :
                              js) :
                 v.js === true?                //ok set js to {}
                   {} :                        //  or to ctx.js
                   v.js;                       //  if(ctx.js &&
                                               //     ctx.js!==true)
          js && (
            (jsParams = {})[
              BEM_.INTERNAL.buildClass(this.block, v.elem)] = js);}
`{"b-bla__e-elt":js}`

        buf.push('<', tag);  //ok

        var isBEM = apply(this._mode = 'bem'); //XXX
Что такое `bem` мода? Является ли контекст блоком или элементом?

        typeof isBEM === 'undefined' &&
          (isBEM = typeof v.bem === 'undefined' ?
            v.block || v.elem :
            v.bem);

        var cls = apply(this._mode = 'cls');
        cls || (cls = v.cls);
Установили класс через моду или взяли прямо из контекста

        var addJSInitClass = v.block && jsParams;
        if(isBEM || cls) {
Теперь будем создавать имя класса.

          buf.push(' class="');
          if(isBEM) {
А если не `isBEM`, то в классе будет только `i-bem` (если это понадобится)

            BEM_.INTERNAL.buildClasses(this.block, v.elem, v.elemMods || v.mods, buf);

            var mix = apply(this._mode = 'mix');
            v.mix && (mix = mix? mix.concat(v.mix) : v.mix);
Миксин делаем здесь! TODO: describe mixin production

            if(mix) {
              var visited = {};

              function visitedKey(block, elem) {
                return (block || '') + '__' + (elem || '');}

              visited[visitedKey(this.block, this.elem)] = true;

              // Transform mix to the single-item array if it's not array
              if (!this._.isArray(mix)) mix = [mix];
              for (var i = 0; i < mix.length; i++) {
                var mixItem = mix[i],
                hasItem = mixItem.block || mixItem.elem,
                block = mixItem.block || mixItem._block || _this.block,
                elem = mixItem.elem || mixItem._elem || _this.elem;

                hasItem && buf.push(' ');
                BEM_.INTERNAL[hasItem? 'buildClasses' : 'buildModsClasses'](
                  block,
                  mixItem.elem || mixItem._elem ||
                    (mixItem.block ? undefined : _this.elem),
                  mixItem.elemMods || mixItem.mods,
                  buf);

                if(mixItem.js) {
                  (jsParams || (jsParams = {}))[BEM_.INTERNAL.buildClass(block, mixItem.elem)] = mixItem.js === true? {} : mixItem.js;
                  addJSInitClass || (addJSInitClass = block && !mixItem.elem);}

                // Process nested mixes
                if (hasItem && !visited[visitedKey(block, elem)]) {
                  visited[visitedKey(block, elem)] = true;
                  var nestedMix = apply(
                    this.block = block,
                    this.elem = elem,
                    'mix');

                  if (nestedMix) {
                    for (var j = 0; j < nestedMix.length; j++) {
                      var nestedItem = nestedMix[j];
                      if (!nestedItem.block &&
                          !nestedItem.elem ||
                          !visited[visitedKey(
                            nestedItem.block,
                            nestedItem.elem
                          )]) {
                        nestedItem._block = block;
                        nestedItem._elem = elem;
                        mix.splice(i + 1, 0, nestedItem);}}}}}}}

          cls && buf.push(isBEM? ' ' : '', cls);

          addJSInitClass && buf.push(' i-bem');
          buf.push('"');
        }
Построим onclick-хак для передачи js-параметров

        if(jsParams) {
          var jsAttr = apply(this._mode = 'jsAttr');
          buf.push(
            ' ', jsAttr || 'onclick', '="return ',
            this._.attrEscape(JSON.stringify(jsParams)),
            '"');
        }
Построим аттрибуты из словаря `ctx.attrs`

        var attrs = apply(this._mode = 'attrs');
        attrs = this._.extend(attrs, v.attrs); // NOTE: возможно стоит делать массив, чтобы потом быстрее сериализовывать
        if(attrs) {
          var name; // TODO: разобраться с OmetaJS и YUI Compressor
          for(name in attrs) {
            if (attrs[name] === undefined) continue;
            buf.push(' ', name, '="', this._.attrEscape(attrs[name]), '"');}}}
Построили тэг. Если это был короткий тэг, тут же его закроем:

      if(this._.isShortTag(tag)) {
        buf.push('/>');
      } else {
        tag && buf.push('>');
А здесь построим контент для полного тэга:

        var content = apply(this._mode = 'content');
мода вернула ctx.content по умолчанию

        if(content || content === 0) {
          var isBEM = this.block || this.elem;
`isBEM` ~ мы внутри шаблона бэм-сущности

          apply(
            this._notNewList = false,
другими словами, `newList=true`

            this.position = isBEM ? 1 : this.position,  
А что было раньше в this.position?

            this._listLength = isBEM ? 1 : this._listLength,
            this.ctx = content,
            this._mode = '');
        }

        tag && buf.push('</', tag, '>');
      }
    }
С главным все!

Теперь можно разобрать работу оптимизации. Поидее, они должны быть
прозрачны для построения своих шаблонов поверх базового, так что это
 параграф со звездочкой.

#### Оптимизации

##### Указатель на данные

    this.ctx && !this._.isSimple(this.ctx) && this.ctx.link: {
      function follow() {
        if (this.ctx.link === 'no-follow') return;

        var data = this._links[this.ctx.link];
        return apply(this.ctx = data);
      }

      if (!cache || !this._cacheLog) return follow.call(this);

      // Get contents that should be cached
      var contents = this._buf.slice(this._cachePos).join('');

      // Move buffer position forward
      this._cachePos = this._buf.length;

      this._cacheLog.push(
        // static cache entry
        contents,

        // dynamic entry
        {
          log: this._localLog.slice(),
          link: this.ctx.link
        }
      );

      var res = follow.call(this);

      // Skip non-cacheable contents
      this._cachePos = this._buf.length;

      return res;
    }


##### Кэш

    cache && this.ctx && this.ctx.cache: {
      var cached;

      // setProperty(obj, ['nested', 'key'], 'value')
      function setProperty(obj, key, value) {
        if (key.length === 0) return;

        if (Array.isArray(value)) {
          var target = obj;
          for (var i = 0; i < value.length - 1; i++) {
            target = target[value[i]];
          }
          value = target[value[i]];
        }

        var host = obj, previous;

        for (var i = 0; i < key.length - 1; i++) {
          host = host[key[i]];}

        previous = host[key[i]];
        host[key[i]] = value;

        return previous;}

      if (cached = cache.get(this.ctx.cache)) {
        // Cache hit
        for (var i = 0; i < cached.log.length; i++) {
          if (typeof cached.log[i] === 'string') {
            this._buf.push(cached.log[i]);
            continue;}

          // Apply log and create reverse movement
          var log = cached.log[i], reverseLog;

          reverseLog = log.log.map(function(entry){
            return {
              key: entry[0],
              value: setProperty(this, entry[0], entry[1])};},
                                   this).reverse();

          // Generate content
          apply(this.ctx.cache = null,
                this._cacheLog = null,
                this.ctx.link = log.link);

          // Revert changes
          reverseLog.forEach(function(entry) {
            setProperty(this, entry.key, entry.value)
          }, this);}
        return cached.res;
      }

      // Record log
      var cacheLog = [],
      res;

      local(this.ctx.cache = null,
            this._cachePos = this._buf.length,
            this._cacheLog = cacheLog,
            this._localLog = []) {
        res = apply();

        var tail = this._buf.slice(this._cachePos).join('');
        if (tail) cacheLog.push(tail);
      }

      // And set cache
      cache.set(this.ctx.cache, { log: cacheLog, res: res });

      return res;
    }












