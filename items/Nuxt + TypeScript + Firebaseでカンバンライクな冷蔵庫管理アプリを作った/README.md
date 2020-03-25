# はじめに
たまに思い立って自分用に何か作って記事を書くがテンプレ化しつつあるフロントエンド人間です。
今回は前々から欲しいと思っていたものの、案がまとまらず放置していたものをやっとそれっぽい形にしました。動機となったテーマは、「冷蔵庫内の食材管理を手軽にしたい」です。

フロントエンドでもはや避けては通れない潮流となっているTypeScriptと、最近勢いが凄まじい[Nuxt](https://nuxtjs.org/)の組み合わせで作ってみました。

# tl;dr
* 冷蔵庫内の食材をカンバン的に管理できるWebサービスを作った
* Nuxt使ってみた
* TypeScript使ってみた
* Firestore使ってみた

https://rich-fridge.firebaseapp.com/public
![Apr-05-2019 21-32-50.gif](https://qiita-image-store.s3.amazonaws.com/0/172228/528d456d-90e0-01b0-dcae-888b8e953bc3.gif)

# 機能
## カンバン
行(レーン)と列(ステージ)のある典型的なカンバン方式で食材（アイテム）の状態を管理する。
レーンは複数作ることができ、冷蔵庫、冷凍庫など好きな区分を設けることが可能。

アイテムの右端をタップすると次のステージに進む。最終ステージからは最初のステージに戻ってくるループ仕様。
「在庫十分 > そろそろ危ない > 在庫切れ」という感じで食材の状態管理が可能。

タスク管理に使えなくもないが、アイテムが持てる情報は少なめなのでtrello使ったほうが幸せになれる。利用シーンの想定はあくまでも食材管理。
![Apr-07-2019 15-54-01.gif](https://qiita-image-store.s3.amazonaws.com/0/172228/26684563-d1e8-5137-a5db-9a95621c240c.gif)

## サブレーン
レーン内でさらにサブレーンを設置可能。「冷蔵庫 > 野菜」、「常温 > 飲料」などの整理ができる。
![Apr-07-2019 18-34-47.gif](https://qiita-image-store.s3.amazonaws.com/0/172228/83d56a5e-0942-eb7a-01ae-e0ab5d41958d.gif)

いくらでもサブレーンをネストしていけたら面白そうな気もするが、データ構造は問題なくてもUI設計が厳しいので１階層のみ。

## テンプレート
おすすめのそれっぽいカンバン構成でさくっと初期化可能。レーンとステージを全て削除すると選択できる。
<img width="332" alt="スクリーンショット 2019-04-07 18.36.10.png" src="https://qiita-image-store.s3.amazonaws.com/0/172228/54c9057b-c1ea-8ff5-645e-c4df6cc0877e.png">

## 公開ファイル
https://rich-fridge.firebaseapp.com/public
認証不要で、URLさえ知っていれば誰でも読み書き可能ファイルを作成可能。見つかっていたずらされる可能性もゼロではないが、さくっと使いたい場合に便利かもしれない。
URLの再発行はできないので、見失ったら探すことはできない。

後からマイページに複製することも可能。
<img width="335" alt="スクリーンショット 2019-04-07 18.37.10.png" src="https://qiita-image-store.s3.amazonaws.com/0/172228/940be2e8-8b94-bdd9-ab8b-3c1ac378bf0f.png">

## 認証マイページ
https://rich-fridge.firebaseapp.com
Google認証でログインが可能（Firebaseが提供してくれる機能をそのまま利用）。
自分だけしか読み書きできないファイルを作成できる。

まだ誰かとファイルを共有することはできない。誰とは言わないが、作者が冷蔵庫を誰かと共有するような生活を送るようになったら機能が作られるかもしれない。

## 推奨端末
モバイルでの利用を前提として作っているのでモバイル端末推奨。
むしろ作者の所有端末であるiPhoneSEの画面サイズでのChrome開発環境でしか動作検証はしていない。

それっぽいPWA対応がされているので、ホーム画面に追加すると快適なアプリライク環境となる（認証後ページのみ）。

## アカウント削除
気が済んだらGoogle認証情報ごとアカウントを削除可能。
作ったデータを全て削除するようにCloud functionsが動き出す。
<img width="343" alt="スクリーンショット 2019-04-07 18.38.45.png" src="https://qiita-image-store.s3.amazonaws.com/0/172228/74a3cd7d-af20-b6e5-03ae-a5fcfed06458.png">

## 注意
Firestoreの無料プラン（Spark）準拠。Documentの読み取りが5万/日なので、それ以上の利用があると多分サービスが止まる。取らぬ狸の皮算用ということでそんな事態はなってから考える。
https://firebase.google.com/pricing/?authuser=0

Google認証をした場合、メールアドレスはFirebaseコンソールから見れる。プライバシーポリシーなどは特に用意していないので、作ったデータと合わせて取り扱いは作者の良心を信じるしかないという前提で。

# 構成
以前Nuxt + TypeScriptを試そうとして大苦戦した記憶があったものの、最新版Nuxtは既にTypeScriptサポートをしてくれているので今回はあまり苦戦しなかった。むしろ後述するビルドスピード改善のほうがよっぽど時間がかかった。
https://nuxtjs.org/guide/typescript/

## Nuxt(2.5.1)
`create-nuxt-app(6.7.0)`にてプロジェクト新規作成後、TypeScriptに対応した最新版のNuxtが使いたかったのでとりあえず全アップグレード。

```sh
$ yarn upgrade-interactive --latest
```

package.jsonはこんな感じ。`@nuxt/typescript`、`vue-property-decorator`あたりは手動で`install`した記憶がある。

```json
{
  ~ 略 ~
  "dependencies": {
    "@fortawesome/fontawesome-svg-core": "^1.2.17",
    "@fortawesome/free-solid-svg-icons": "^5.8.1",
    "@fortawesome/vue-fontawesome": "^0.1.6",
    "@nuxtjs/axios": "^5.4.1",
    "@nuxtjs/pwa": "^2.6.0",
    "cross-env": "^5.2.0",
    "firebase": "^5.9.1",
    "nuxt": "^2.5.1",
    "nuxt-fontawesome": "^0.4.0"
  },
  "devDependencies": {
    "@nuxt/typescript": "^2.5.1",
    "@nuxtjs/eslint-config": "^0.0.1",
    "@types/jest": "^24.0.11",
    "@typescript-eslint/eslint-plugin": "^1.5.0",
    "@vue/test-utils": "^1.0.0-beta.27",
    "autoprefixer": "^9.5.0",
    "babel-core": "7.0.0-bridge.0",
    "babel-eslint": "^10.0.1",
    "babel-jest": "^24.5.0",
    "eslint": "^5.15.3",
    "eslint-config-prettier": "^4.1.0",
    "eslint-config-standard": ">=12.0.0",
    "eslint-loader": "^2.1.2",
    "eslint-plugin-import": ">=2.16.0",
    "eslint-plugin-jest": "^22.4.1",
    "eslint-plugin-node": ">=8.0.1",
    "eslint-plugin-nuxt": ">=0.4.3",
    "eslint-plugin-prettier": "^3.0.1",
    "eslint-plugin-promise": ">=4.0.1",
    "eslint-plugin-standard": ">=4.0.0",
    "eslint-plugin-vue": "^5.2.2",
    "jest": "^24.5.0",
    "nodemon": "^1.18.10",
    "prettier": "^1.16.4",
    "tailwindcss": "^0.7.4",
    "ts-jest": "^24.0.0",
    "vue-jest": "^3.0.4",
    "vue-property-decorator": "^8.1.0"
  }
}
```

たまには`yarn`でなくて素の`npm`でも使ってみようかなと思ってしばらく使ってみたが、`yarn upgrade-interactive`に似たことをしたくて`npm-check`というパッケージをインストールしたあたりで、「これ`yarn`インストールすればよくない？」と気づいてしまって試みは終了。

### TypeScript

#### tsconfig.json
`{}`とだけ書いた`tsconfig.json`を設置した状態でNuxtを起動すると、TypeScriptを利用したいことを勝手に汲み取ってくれて必要な設定を`tsconfig.json`に書き込んでくれる。
TypeScript化に必要なパッケージが未インストールだと色々とエラーが出るので追加していく（何を入れたかは失念）。

追加で、Vueコンポーネントを[vue-property-decorator](https://github.com/kaorun343/vue-property-decorator)で書けるように設定を調整。

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "allowJs": true,
    "paths": {
      "~/*": [
        "./*"
      ],
      "@/*": [
        "./*"
      ]
    },
    "target": "esnext",
    "module": "esnext",
    "moduleResolution": "node",
    "lib": [
      "esnext",
      "esnext.asynciterable",
      "dom"
    ],
    "esModuleInterop": true,
    "sourceMap": true,
    "strict": false,
    "noImplicitAny": false,
    "noEmit": true,
    "allowSyntheticDefaultImports": true,
    "types": [
      "@types/node",
      "@nuxt/vue-app",
      "jest",
      "./types"
    ]
  }
}
```

#### .eslintrc

```js
module.exports = {
  root: true,
  env: {
    browser: true,
    node: true,
    'jest/globals': true
  },
  parserOptions: {
    parser: '@typescript-eslint/parser',
    ecmaFeatures: {
      legacyDecorators: true
    },
    project: './tsconfig.json'
  },
  extends: [
    '@nuxtjs',
    'plugin:nuxt/recommended',
    'plugin:prettier/recommended',
    'prettier',
    'prettier/vue'
  ],
  plugins: ['prettier', '@typescript-eslint', 'jest'],
  rules: {
    'nuxt/no-cjs-in-config': 'off',
    'no-console': 'off',
    'vue/attribute-hyphenation': ['error', 'never']
  }
}
```

#### Nuxtのビルドが遅すぎる問題
これまでNuxtをさくっと試すたびに思っていたのだが、ビルドが遅すぎる。初回起動に時間がかかるのは仕方ないにしても、`watch`モードの差分ビルドでも時間がかかりすぎる。
まるっと全部いい感じにしてくれるNuxtの黒魔術故にそんなものなのかとも考えていたが、ちょっといじったスタイルが反映されるのに毎回1分近く待たされるのはさすがに許容できる範囲ではなかった。

どこで時間がかかっているのか調べまわって判明した犯人はEslint。`nuxt.config.js`からEslint部分をコメントアウトしたら差分ビルドも1~2秒で終わるようになってやっと人権のある開発環境となった。
lintエラーを見逃してしまう可能性は上がるが、prettier入ってるし保存するときに自動整形する設定しておけばそんな見逃すこともないだろうということで解除。

おそらく不要なファイルまでEslint対象に含まれてしまっているとかだろうからそのあたりの設定を見直すべきなのだろうが、`.eslintignore`を少し弄ってみても改善できなかったので今後の課題として今は妥協。

```js
  /*
   ** Build configuration
   */
  build: {
    /*
     ** You can extend webpack config here
     */
    extend(config, ctx) {
      // Run ESLint on save
      // build速度が20倍くらい違うので外す
      // if (ctx.isDev && ctx.isClient) {
      //   config.module.rules.push({
      //     enforce: 'pre',
      //     test: /\.(js|vue)$/,
      //     loader: 'eslint-loader',
      //     exclude: /(node_modules)/
      //   })
      // }
    }
  }
```

### Jest
最近は流行に乗っかりテストはJestを採用。

#### jest.config.js
TypeScript対応をしたくらいで`create-nuxt-app`の初期状態からそれほど変更はなし。

```js
module.exports = {
  setupFiles: ['<rootDir>/test/jest.setup.js'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/$1',
    '^~/(.*)$': '<rootDir>/$1',
    '^vue$': 'vue/dist/vue.common.js'
  },
  moduleFileExtensions: ['js', 'ts', 'vue', 'json'],
  transform: {
    '^.+\\.jsx?$': 'babel-jest',
    '^.+\\.tsx?$': 'ts-jest',
    '^.+\\.vue$': 'vue-jest'
  },
  globals: {
    'ts-jest': {
      tsConfig: 'tsconfig.json'
    }
  },
  testRegex: '(/__tests__/.*|(\\.|/)(test|spec))\\.(jsx?|tsx?)$',
  testPathIgnorePatterns: [
    '<rootDir>/node_modules/',
    '<rootDir>/.nuxt/',
    '<rootDir>/coverage/'
  ],
  collectCoverage: false,
  collectCoverageFrom: [
    '<rootDir>/components/**/*.vue',
    '<rootDir>/pages/**/*.vue'
  ]
}
```

#### Dockerコンテナ内でJestが遅すぎる問題
Nuxtビルドが遅い問題は完全にEslint起因だったが、Jestは最初からEslintをかけていなかったにも関わらず実行がとんでもなく遅かった。`watch`モードで差分ビルドをするときでも半端なく遅く、１ファイルごとに20秒くらいかかっているような状況だった。

この問題はDocker環境起因らしく、公式に書いてあった`--runInBand`オプションを付ける対応をしたら解決。
https://doc.ebichu.cc/jest/docs/ja/troubleshooting.html#docker-ci

### [Tailwind CSS](https://tailwindcss.com/docs/what-is-tailwind/)
nuxt-create-appでプロジェクト作成するときに選択肢にあったのでそのまま採用。
最初は[Vuetify](https://vuetifyjs.com/ja/)を使うつもりだったが、TypeScript化したらコンソールにエラーが表示されるようになって解決できなかったので噂のTailwind CSSにチャレンジ。

Tailwind CSSはスタイルを簡単に指定できる`class`を提供してくれるだけでコンポーネントは含まれていない。ただし豊富なサンプルもあるので特に不便は感じなかった。むしろ本当に欲しかったものの最小構成感が心地よい。
https://tailwindcomponents.com/

その特性上`class`指定が多くなり、`<style>`を書くことがかなり減った。そしてせっかくなので今回は`<style>`を捨ててCSS in JSな書き方に寄せてみた。
一般にどちらがいいかの結論は出せないが、Tailwind CSSを使うなら`class`と`<style>`にスタイル情報が分断されるよりは、両者を近い場所に書けるCSS in JSの方が見やすそう。

とはいえ完全に`<style>`を捨てることはできず、`<transition>`利用のスタイル指定などは`<style>`で行っている。

Tailwind CSSと[fontawesome](https://fontawesome.com/)があればデザイン力が貧弱な個人開発者の悩みは８割方解決するのではなかろうか。fontawesomeをNuxtにいい感じに導入する方法も整備されているのでさくっと使い始められる。
https://qiita.com/OvThAlmin/items/154e53952bc46d91d44c

### PWA
`create-nuxt-app`でPWAを希望すれば全部いい感じに準備してくれる。ホーム画面に追加したらもちろん`standalone`で快適に使えるし、Service Workerでキャッシュもしてくれる。
アイコン類は変えたかったので、`static/`に入っていた`icon.png`と`favicon.ico`を差し替えて終わり。

例によってデザイン力はないのでロゴどうしようかなと悩んでいたらこんな記事が目に入って神サービスを発見。
https://note.mu/rdlabo/n/n9a8f8391529c

適当にポチポチしていくと適当な感じのアイコンが作れてしまって最高の体験だった。
![D3hwAx0U8AEmAeV.png](https://qiita-image-store.s3.amazonaws.com/0/172228/f883f288-d5ef-afe8-6ac0-83f7b947d566.png)

## Firebase
何かWebサービスを作りたくなってしまったフロントエンドエンジニアに最高のBaaSを提供してくれる頼れる存在。

当初はCloud functionsからNuxtを使ってSSRしてみようかと目論んでいたが、SSRの必要性がなさすぎて頓挫。Nuxtを`mode: 'spa'`にしてビルドしたアセットをFirebaseにhostingするというシンプルなデプロイとなっている。
ただSSRしないにしても、ルーティングやミドルウェアなどNuxtの恩恵を大いに受けられることは変わりない。PWAのいい感じの設定まで用意しておいてくれるのはホスピタリティが高すぎる。

### Firestore
使い慣れたRealtimeDBでいいかとも考えたが、Firebaseが激しく推していしせっかくなので新しいものにチャレンジ。
DocumentとCollectionを中心としたら構造に最初は慣れないが、分かってくるとRealtimeDBでは実現できなかった表現力の可能性に気づき始める。

ただしRealtimeDBが通信量ベースの従量課金だったのに対し、FirestoreはDocumentの読み書きベースに変更されている点は要注意。Document数なので、一覧データを取得したりするとその要素数分がカウントされていく。
個人でだけ使っている分には無料枠をはみ出る心配はそれほどないが、ある程度の利用量を期待していく場合は料金シミュレートは慎重にやったほうが良さそう。

データ保護の権限設定もRealtimeDBより格段に表現力と書きやすさが向上している。
ただし権限判定のために行うDocumentの読み取りでも従量課金対象としてカウントされていくのが地味に怖い。複雑な権限判定を設定していくと、読み取り回数がどんどん掛け算されていくことになる。

権限設定で不便だなと思ったのは、判定情報としてCollectionからクエリで取得したDocumentを使えないこと。関係テーブルみたいなCollectionを用意しておいて、`where('onwer', '==', id)`みたいな感じで権限を検索してという素朴な実装は成り立ちそうにない。

Firestoreはまだベータだが、Firebaseが推しまくっているように確かにそのポテンシャルはRealtimeDBを凌駕しているのは間違いない。正式リリースに向けて今後もバシバシ素晴らしい機能が追加されていくことを期待して応援。

### Cloud functions
Firestoreの仕様で、Documentを削除してもその配下にあるCollection（サブコレクション）は削除されない。あくまでパスが重なっているというだけであって、データとして全く独立した存在らしい。
FirebaseコンソールからDocumentの削除をした場合はちゃんと配下のCollectionも一括で消してくれる。

クライアント側でサブコレクションも消すようにすればいいかとも考えたが、ルートから辿れないサブコレクションはFirebaseコンソールでもパスを直打ちしないと見えないようになってしまってなんだか不安なので、Cloud functionsを使ってゴミが残りにくいように頑張っておく。

FirestoreのドキュメントでもCollectionの一括削除をクライアント側でやるのは非推奨となっているので素直に従う。
https://firebase.google.com/docs/firestore/manage-data/delete-data?hl=ja

サブコレクションを全部消したいというシチュエーションはかなりあると思うので、Firestore側にそういう機能が追加されることを期待したい。
