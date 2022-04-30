# 正規表達式

## 宣告方式

* literal: 不能動態, 只能是固定的字串, 使用在知道內容不會有變化的情況, 效能較好.
* constructor: 可以是動態的, 使用在知道內容會有變化或是還不知道內容（例如從其他地方取得的字串）的情況.

### literal

宣告方式用斜線（/）夾住字串, 例如：`var re = /ab+c/;`.

### constructor

用`RegExp`建立實體, 例如：`var re = new RegExp('ab+c');`.