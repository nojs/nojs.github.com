<link rel="stylesheet" href="/css/markdown.css"></link>

## Супер-шаблонизатор!

Отправляюсь вместе с вами в путешествие. Сейчас, когда я пишу эти
строки, bemhtml кажется мне далеким и неизведанным островом, покрытым
таинственными и опасными джунглями.

### Шаблон по-умолчанию

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

контекст установлен, но не булев -- положим его в буфер:

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
И итерация!

      while(i < l)
        apply(this.ctx = v[i++]);
восстановим `.position`, если не `prevNotNewList`:

      prevNotNewList || (this.position = prevPos);
    }
Теперь случай по-умолчанию:

    true: {
      var vBlock = this.ctx.block,
      vElem = this.ctx.elem,
Для чего нужны vThing?

      block = this._currBlock || this.block;
      this.ctx || (this.ctx = {});
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













