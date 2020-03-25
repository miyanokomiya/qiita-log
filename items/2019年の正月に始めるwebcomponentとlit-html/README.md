## 前置き

https://qiita.com/aggre/items/3ed2558a9fb8ba887385
こちらの記事を大いに参考にさせてもらいました。被っている内容もあります。
この分野周辺の変遷が早すぎるせいか変わっている部分がそこそこあったので、動かすための備忘録です。

## コンポーネント全容
とりえあず動かせたコンポーネントの全体像。

* html側

```html
<x-product name="JavaScript" src="imageSrc" />
```
* js側

```js
import { html, render as litRender } from 'lit-html'

function render (h, c) {
  return litRender(h, c.shadowRoot || c.attachShadow({ mode: 'open' }))
}

class XProduct extends HTMLElement {
  static get observedAttributes () { return ['name'] }
  constructor () { super() }

  html (name, src) {
    return html`<img alt="${name}" src="${src}" @click="${() => this.onClick()}" />`
  }

  connectedCallback () {
    const name = this.getAttribute('name')
    const src = this.getAttribute('src')
    render(this.html(name, src), this)
  }

  attributeChangedCallback (attributeName: string, oldValue: string, newValue: string, namespace: string) {
    console.log('attributeChangedCallback', attributeName, oldValue, newValue, namespace)
  }

  onClick () {
    console.log(this.getAttribute('src'))
  }
}
customElements.define('x-product', XProduct)

```

## ポイント
### constructor内でattributeは参照できない
2017年から既出っぽいが、webcomponentの使い方で調べると`constructor`内で`getAttribute`している内容が多い。
何か歴史的変遷があったのかもしれない。とりあえず現時点では取得できない。

`connectedCallback`内であれば取得できるのでそちらで初期化する。

```js
  connectedCallback () {
    const name = this.getAttribute('name')
    const src = this.getAttribute('src')
    render(this.html(name, src), this)
  }
```

https://stackoverflow.com/questions/42719914/custom-element-class-this-getattributedata-returns-null

### イベントハンドラは`@`
```js
html`<img alt="${name}" src="${src}" @click="${() => this.onClick()}" />`
```
[以前](https://qiita.com/aggre/items/3ed2558a9fb8ba887385#vanilla-web-components--lit-html)は拡張機能扱いでまだ仕様が揺れていたようだが、今は標準機能入りしてVue.jsっぽい定義方法になっている。

https://lit-html.polymer-project.org/guide/writing-templates#add-event-handlers

### `attributeChangedCallback`の監視対象は明示する必要がある
`observedAttributes`という`static`メソッドで`return`された属性名のみが`attributeChangedCallback`の監視対象となる。
マウント時点で監視対象属性に値が入っている場合、`connectedCallback`より前に`attributeChangedCallback`が実行される。

```js
static get observedAttributes () { return ['name'] }

attributeChangedCallback (attributeName: string, oldValue: string, newValue: string, namespace: string) {
  console.log(attributeName, oldValue, newValue, namespace)
}
```

https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements#Using_the_lifecycle_callbacks

### shadow domの初期化は一度だけ
`attachShadow`を２回実行するとエラーになるので、２回目移行は`shadowRoot`を参照する必要がある。
先人に倣って関数に切り出したが、`constructor`で`attachShadow`しておき、後は`shadowRoot`を使うという風にするだけでいいかもしれない。

```js
function render (h, c) {
  return litRender(h, c.shadowRoot || c.attachShadow({ mode: 'open' }))
}
```

https://qiita.com/aggre/items/3ed2558a9fb8ba887385#%E3%82%A2%E3%83%83%E3%83%97%E3%83%87%E3%83%BC%E3%83%88%E3%81%AB%E5%AF%BE%E5%BF%9C%E3%81%99%E3%82%8B

## おまけ
### es5トランスパイルは不可
es2015のclass機能が必要なので、es5へトランスパイルしているとエラーになる。
es2015非対応ブラウザがwebcomponentに対応しているわけがないので潔く切る。

```
Failed to construct 'HTMLElement': Please use the 'new' operator, this DOM object constructor cannot be called as a function.
```
