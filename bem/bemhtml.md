<link rel="stylesheet" href="/css/markdown.css"></link>

## Супер-шаблонизатор!

Отправляюсь вместе с вами в путешествие. Сейчас, когда я пишу эти
строки, bemhtml кажется мне далеким и неизведанным островом, покрытым
таинственными и опасными джунглями.

### Шаблон по-умолчанию

создадим контекст шаблона

{
  {
  
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
  }
}






