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
Очень хитро!

      if(prevNotNewList) {
        this._listLength += l - 1;
      } else {
        this.position = 0;
        this._listLength = l;
      }

      this._notNewList = true;

      while(i < l)
        apply(this.ctx = v[i++]);

      prevNotNewList || (this.position = prevPos);
    }











