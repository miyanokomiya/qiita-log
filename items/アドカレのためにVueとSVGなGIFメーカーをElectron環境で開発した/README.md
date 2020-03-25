# 前説
どうも年一SVG芸人です。前回投稿から早一年経とうとしてますが、またVueとSVGでそれっぽい何かを作ったので紹介と、開発で遭遇した問題のポストモーテム的なものをまとめていきます。

今回はElectronにも手を出してみました。

# 作ったもの
## 概要
* github  
https://github.com/miyanokomiya/svgif
* web版  
https://svgif-c077b.firebaseapp.com/
* electron版  
未公開（リポジトリクローンしてyarn devすれば使える！）

GIFメーカーです。自産自消でさっそくそのGIFメーカーを使って作った概要GIFがこちらです。
![サンプル.gif](https://qiita-image-store.s3.amazonaws.com/0/172228/1e366e29-6907-8ba1-fdd9-5de7edb583ce.gif)

### 機能
* 背景画像ベースのタイムライン
* 矩形、矢印、文字、画像の挿入と変形
* ↑の履歴操作
* タイムラインレイヤー管理
* gifの個別フレーム設定
* gifプレビュー再生
* gif生成（容量が大きくなるとダウンロードボタンが機能しないので、右クリックから保存推奨）
* 編集データの書き出しと読み込み

## 動機
定期的に湧いてくるSVGで何か作りたい欲に抗うことができず、たまたま[gif.js](https://jnordberg.github.io/gif.js/)というものが目に入ってしまった結果として手が動き出したのが発端です。
さらにたまたまElectronが目に入ってしまったがために無駄にElectron環境で作りました。

なお今はgo言語に呼ばれている気がしてきてしまったので、このプロダクトは一旦切りを付けてしばらくgoの世界を探検してきます。

## 構成など
### [Vue](https://jp.vuejs.org/index.html)
VueのアドベントカレンダーなのでもちろんVueで作っています！
そしてもちろんみなさん既に当然ご存知のようにVueとSVGの組み合わせは相性ばっちりで、宣言的なSVG制御は今後のスタンダードとなっていくこと間違いなしでしょう。

え？[**ご存じない？**](https://qiita.com/miyanokomiya/items/3d195d8b6e09097d0102)

### [Electron](https://electronjs.org/)
vue-cliに[Electronボイラープレート](https://github.com/SimulatedGREG/electron-vue)があったのでそちらを利用しています。
しかしこのボイラープレートがなかなか癖が強く苦戦した箇所がちらほら・・後のポストモーテム集で紹介していきます。

### SVG
キャンバス部分の表示や図形を書いたりする部分はSVGで作っています。

* 矩形
矩形編集を作っておけば円や画像にもそのまま使えて便利です。
編集はこんな感じ。
![rec.gif](https://qiita-image-store.s3.amazonaws.com/0/172228/3c183822-0f68-38df-2edf-133bdbd639b5.gif)

* 矢印
![arrow.gif](https://qiita-image-store.s3.amazonaws.com/0/172228/c24d53f5-00d7-c744-a225-c46485e2b600.gif)

オールSVGで完結するかと思いきや、gif.jsに食わせるためにはcanvasに描画する必要があったのでcanvas要素も若干含まれています。
ただ（一部問題はあったものの）SVGはそのままcanvasに描画できたので、登場するのはほぼSVGのみです。

### [Vue Material](https://vuematerial.io/)
見た目部分は楽したかったこともあり、コンポーネントフレームワークとしてVue Materialを利用しています。レイアウトは揃っているので面倒なアンカー処理とかせずに済むので助かります。

今まで[Vuetify](https://vuetifyjs.com/ja/)を使うことが多かったんですが、たまには違うものも使ってみようと思いなんとなく使っています。
そこまで使い込んでいはいないので優劣は着けづらいですが、さくっと使う分にはどちらでも特に遜色はないかなという印象です。

ただVue Materialは用意されているアイコンが少なく、拡張の仕方もよく分からなかったので、その点についてはVuetifyの方が揃っていて便利だったかなと思います。

### [Firebase](https://firebase.google.com/?hl=ja)
Electronで作ったものの公開するにはやっぱりweb上で見れないと不便だよねということで頑張ってwebサイトとしてビルドしてFirebaseにホスティングしました。

今回はローカルなアプリケーションとして完結させたかったので、サーバーとデータのやりとりはしていません。
編集データをサーバーに保存したりということもやっていないですが、もし不安であればネットワーク遮断した上での利用をおすすめします。


ただ静的サイトをホスティングするためだけにFirebaseさんを使うのは若干無駄使い感もありましたが、やり方を知ってて一番手っ取り早くて安上がりな方法だったので贅沢に使わせていただきます。

# できなかったこと
## 画面キャプチャ
じつは当初予定では、というよりElectronを使うならやってみたかったものの結局まとまらずに投稿日を迎えてしまったものがあります。

ずばり画面キャプチャです。フロントエンドエンジニアのみなさんなら、「ブラウザ上でキャプチャしたい」という無茶要望を数千回と聞いてきたことでしょう。
しかしElectronという環境でなら、ブラウザを超えた世界を見ることができるので画面キャプチャを実装できない理由はもはやありません。

そして実際にキャプチャができるそれっぽいものは作ったのですが、どうしても超えられない大きな問題が立ちふさがりました。
なんとElectronでは、**windowの一部だけをクリック透過させることはできない**という制限があるらしいのです。

<img width="627" alt="スクリーンショット 2018-12-02 17.02.07.png" src="https://qiita-image-store.s3.amazonaws.com/0/172228/6541966c-97c9-6094-4c67-cd32773dbae9.png">
こちらが実際に作ったそれっぽいキャプチャ用windowなんですが、残念なことにこれを全面に配置すると背景を触ることができなくなります。。。
他にも背景透過するとヘッダーも透過されてしまったりとまだまだ透過まわりには不便が多いみたいです。

複数windowを組み合わせて一部透過しているような単一っぽいwindowをでっちあげるとか対処法はありそうですが、これだけでも重そうだったので今回のスコープからは泣く泣く削除となりました。

いつかリベンジしたいと思いつつ、もし既にいい感じに実装している例などありましたら是非とも教えてください。

## モバイル対応
「Electronを使うんだから仕方ない」精神のもと、早々にモバイル対応は諦めました。
久しぶりにデスクトップ特化なUIを考えていったんですが、モバイルを諦めた瞬間から世界は大きく広がると実感しました。
デスクトップでどっしりちまちまやりたいような領域では、マウス操作に特化したUIという選択もまだまだありなんだろうなとしみじみしていました。

# ポストモーテム集
以下、ポストモーテムという名の詰まったこと集です。
Vue + Electron + SVGで何かを作りたくなってしまった方々の助けに少しでもなってもらえたら詰まってた時間もうかばれてくれます。

## VueとElectron
実はElectronをまともに触るのは初だったのこともあり、詰まったことの大部分はElectron周りでのものでした。

特に[vuex-electron](https://github.com/vue-electron/vuex-electron)という如何にも便利そうなプラグインを入れたことによる予期せぬ挙動が多かったです。
vue-cliでこれを使うかどうかの選択肢が出るのですが、どういうものかよく分かっていないうちは避けておくことをおすすめします。

### actionとコンポーネントの描画タイミング
#### 現象
* 同一スレッドで複数のactionをdispatchすると、更新前の状態のコンポーネントがちらついて見える。

#### 例
* 図形をドラッグで移動し、マウスアップのタイミングで移動確定actionをdispatchする。
* 単独actionなら違和感なし。
![Nov-04-2018 11-15-27.gif](https://qiita-image-store.s3.amazonaws.com/0/172228/838b4ae3-2053-3d93-1793-44dc826ccd44.gif)
* 複数actionだとちらつく（ちらつかないときがあるのはgifのコマ落ちのため）。
![Nov-04-2018 11-18-07.gif](https://qiita-image-store.s3.amazonaws.com/0/172228/0cb84f66-42a3-8108-da72-dd894e903258.gif)

#### 原因
* 同一スレッドで非同期処理を含むactionを複数dispatchすると、非同期処理が終わる度にstateの状態が更新される。
* state更新の度にcomponentは更新されるため、全ての変更が反映されきっていない状態で都度描画されてちらつく。

**しかし今回の場合、actionに非同期処理は一切書かれていなかった**

#### 真の原因
vuex-electronのsharedMutationはstateを全プロセスで共有するために、action内でプロセス間通信が挟まってからmutationがcommitされる。actionに非同期処理を入れた覚えがなくても必ず非同期処理になってしまうという罠。

#### 対処方法
* 同一スレッドで複数のactinを実行しない or 単独actionで済むよう処理をまとめる。  
* 今回のような場合、オブジェクト（矩形のやつ）の移動を確定するactionを１件ずつループでdispatchすることを避け、複数オブジェクトの移動を一発で確定するactionをdispatchする。
* mutationをこんな感じにし、単独と配列どちらでも更新できるようにした。

```javascript
[types.m.UPDATE_SVG_ELEMENT](state, { svgElement, svgElementList }) {
  svgElementList = svgElementList || [svgElement]
  svgElementList.forEach(svgElement => {
    // svgElementの更新処理
  })
}
```

#### 所感
非同期処理が含まれているactionを同一スレッドで複数dispatchすると起こりうる現象なのだが、どう見てもactionに非同期処理なんて書かれていなかったので原因発見に時間がかかってしまった。真犯人はvuex-electron。

### ビルドエラー for web ①
#### 現象
`yarn build:web`コマンドを実行すると、`fsモジュールが存在しない`というエラーになってしまう。

#### 原因
`vuex-electron`が`fs`参照している。

#### 対処方法
web用のwebpack設定ファイルに以下を追加する。

```javascript
node: {
  fs: 'empty'
}
```

#### 所感
またもvuex-electron

### ビルドエラー for web ②
#### 現象
`yarn build:web`コマンドを実行すると、`ERROR in unknown: Unexpected token`というエラーが起きてバンドルファイルが生成されない。

#### 原因
`BabiliWebpackPlugin`がうまく動いていないっぽい。

#### 対処方法
web用のwebpack設定ファイルにて`plugins`から`BabiliWebpackPlugin`を削除する。

#### 所感
vue-cliのelectronボイラープレートそのままの状態ではこのエラーは起きなかったので、本当の理由は別のところにあるのかもしれない。
エラーメッセージから得られる情報が少なすぎるためとりあえず動いたこの方法で妥協。

### ビルドエラー for web ③
#### 現象
ビルドしたindex.htmlを開くと、`Uncaught TypeError: n.existsSync is not a function`というエラーが起きる。

#### 原因
`'electron'`からモジュールを`import`している。

#### 対処方法
ファイルのトップレベルではなく、`'electron'`モジュールを使う関数内で`require('electron')`する。
トップレベルで必要な場合は`process.env.IS_WEB`を使ってwebビルドかどうか分岐して`require`する。

#### 所感
ファイルのトップレベルで`import`や`require`しているとwebpackが親切にバンドルしてくれてしまう。
vue-cliのelectronボイラープレートではwebビルド判定値を`process.env.IS_WEB`に入れてくれるので活用する。

さらに一部のプラグイン(vuex-electron)でも同様の現象が起こるので、使うかどうかを分岐する必要がある。
vue-cliのelectronボイラープレートでvuex-electronをonにしてwebビルドすると、デフォルトのままでもブラウザでエラーになっているトラップあり。

こんな感じでvuex-electronの利用を分岐すればよい。

```javascript
plugins: process.env.IS_WEB
  ? []
  : [
      require('vuex-electron').createPersistedState(),
      require('vuex-electron').createSharedMutations()
    ],
```

↓PR出してみた。
https://github.com/SimulatedGREG/electron-vue/pull/751

## SVG
### svgのアスペクト比が固定されてしまう
#### 現象
#### 例
* タイムライン部分は時間に応じて引き伸ばして表示したいが、svgで作った赤枠のアスペクト比が崩れない。
<img width="643" alt="スクリーンショット 2018-11-04 15.40.46.png" src="https://qiita-image-store.s3.amazonaws.com/0/172228/08048e96-541b-fa85-9619-ce040fcfb523.png">

#### 原因
* svgにはpreserveAspectRatioというアスペクト比に関する属性があり、デフォルトだとアスペクト比が崩れないようになっている。

#### 対処方法
* preserveAspectRatio="none"という属性をsvgタグに付ける。  

```haml
<svg preserveAspectRatio="none"></svg>
```

* svg部分もアスペクト比を崩して引き伸ばされるようになる。
<img width="630" alt="スクリーンショット 2018-11-04 15.40.57.png" src="https://qiita-image-store.s3.amazonaws.com/0/172228/b62a6d7f-7659-c468-c835-ac422c250561.png">

#### 所感
アスペクト比に関するsvgとcssの挙動。

### `<image>`を含んだSVGをImageのsrcとして指定できない

#### 現象
gif.jsに食わせるために、SVGをImage化してcanvasにdrawImageする必要があった。
SVG文字列をImageとして読み込む処理はこちら。

```js
const DOMURL = self.URL || self.webkitURL || self
const img = new Image()
const svg = new Blob(['SVG文字列'], {
  type: 'image/svg+xml;charset=utf-8'
})
const url = DOMURL.createObjectURL(svg)
img.onload = () => {
  // 好きな処理
}
img.src = url
```

この処理でImage化は成功したと思いきや、SVGに`<image>`タグが含まれている場合は`img.src = url`の部分でエラーになってしまっていた。

```html
<image x="0" y="0" width="100" height="100" xlink:href="画像URLやbase64"  />
```

このエラーはブラウザ環境でのみ発生し、Electron環境では発生しない。
なのでクロスオリジン制約に引っかかるよくあるパターンを疑ったが、`<image>`タグでの画像URL、SVG画像URLを無駄にサーバー経由で取得するようにしても改善せず。

#### 原因
`xmlns:xlink="http://www.w3.org/1999/xlink"`という属性が`<image>`タグに付けられていない。
**Electron環境ではなぜか勝手にこの属性が付与される。**

#### 対処方法
`xmlns:xlink`属性を<image>`タグに付ける。

```html
<image x="0" y="0" width="100" height="100" xlink:href="画像URLやbase64" xmlns:xlink="http://www.w3.org/1999/xlink" />
```
#### 所感
発生する例外のメッセージが空で情報が少なすぎるために原因特定に時間がかかってしまった。
Electron環境で生成したSVG文字列と、ブラウザ環境で生成したSVG文字列を見比べてみてやっとどこに差分があるか気づいて対処方法発見に至る。

繰り返すが、**Electron環境ではなぜか勝手にこの属性が付与される。**

画像を無駄にサーバーから返すために、firebaseのstorageを初めて使ったという若干のプラス要素。無料プランだと容量が心もとないが、サーバーレスな使い心地が素晴らしい。
