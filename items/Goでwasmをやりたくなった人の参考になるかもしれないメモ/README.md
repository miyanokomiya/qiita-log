## 経緯箇条書き
* フロントエンドエンジニアだけどGoを書きたい
* wasmやれば全てが解決するんじゃないかと軽い気持ちで手を出した
* 想像以上に資料が少ない
* ひたすら試行錯誤
* もしかしたら誰かの役にたつかもしれないからまとめておく

## 注意
* Goのwasmは始まったばかりでこれからも急激に変化していくと思われる
* この記事の内容は2018年12月時点での情報
* goのバージョンは`go1.11.4`

## Go側でjsを扱う
### 前置き
大体ここに書いてある
https://medium.zenika.com/go-1-11-webassembly-for-the-gophers-ae4bb8b1ee03

### 汎用
* グローバル取得  

```go
global := js.Global()
```
* 関数実行  

```go
fn.Invoke(引数1, 引数2, ...)
```
* undefined取得  

```go
js.Undefined()
```
* null取得  

```go
js.Null()
```
* bool変換  

```go
value.Bool()
```
* int変換  

```go
value.Int()
```
* float64変換  

```go
value.Float()
```
* string変換  

```go
value.String()
```

### オブジェクト
* オブジェクトを作る  

```go
obj := js.Global().Get("Object").New()
```
* オブジェクトのプロパティ取得  

```go
obj.Get("プロパティ名")
```
* オブジェクトのプロパティ設定  

```go
obj.Set("プロパティ名", 値)
```
* オブジェクトの関数実行  

```go
obj.Call("関数名", 引数1, 引数2, ...)
```
`Get`で関数取得して`Invoke`だと`this`が`obj`ではなくなると思われる（未検証）

```go
obj.Get("関数名").Invoke(引数1, 引数2, ...)
```

### 配列
* 配列を作る  

```go
array := js.Global().Get("Array").New()
```
* 配列に追加  

```go
array.Call("push", 値)
```
* 配列に追加（インデックス指定）  

```go
array.SetIndex(インデックス, 値)
```
* 配列から削除  

```go
array.Call("splice", インデックス, 1)
```
* 配列から取得  

```go
array.Index(インデックス)
```
* 配列の長さ  

```go
array.Get("length")
```
* 配列ループ  

```go
for i := 0; i < array.Get("length"); i++ {
  value := array.Index(i)
}
```

## goとjsの連携
hello worldまではこちらを参考に
https://qiita.com/t29wuQ/items/eacd113ee25bc46b3bd0

それっぽい形になったときのタグ
https://github.com/miyanokomiya/okaphy/tree/wasm_sample

### やりたいこと
* jsからgoの関数に引数を渡して実行する
* goの関数からjs側に戻り値を返す

### 注意
* 最適な方法とは限らない
* 全体的に参考にできる資料が少ないので試行錯誤実装

### go側

```go
package main

import (
	"fmt"
	"syscall/js"
)

var goWasm = js.Global().Get("GoWasm")

func main() {
	c := make(chan struct{}, 0)

	wasmWrapper("echo", echo)

	goWasm.Call("onLoad")
	<-c
}

func wasmWrapper(name string, fn func(value js.Value) (interface{}, error)) {
	handler := js.NewCallback(func(values []js.Value) {
		fmt.Println("exec " + name)

		if len(values) == 0 {
			var val js.Value
			fn(val)
			return
		}

		arg := values[0]

		data := arg.Get("data")
		res, err := fn(data)

		if err != nil {
			fail := arg.Get("fail")
			if fail != js.Undefined() {
				fail.Invoke(err.Error())
			}
			return
		}

		done := arg.Get("done")
		if done != js.Undefined() {
			done.Invoke(res)
		}
	})

	goWasm.Get("functions").Set(name, handler)
	goWasm.Call("onAddFunction", name)
}

func echo(value js.Value) (interface{}, error) {
	return value, nil
}
```

### js側

```js
window.GoWasm = {
  functions: {},
  onAddFunction(name) {
    console.log('add wasm function: ', name)
  },
  onLoad () {
    console.log('wasm functions loaded')
  },
  init () {
    if (!WebAssembly.instantiateStreaming) { // polyfill
      WebAssembly.instantiateStreaming = async (resp, importObject) => {
        const source = await (await resp).arrayBuffer
        return await WebAssembly.instantiate(source, importObject)
      }
    }

    const go = new Go()
    let mod, inst
    WebAssembly.instantiateStreaming(fetch("main.wasm"), go.importObject)
      .then(result => {
        mod = result.module
        inst = result.instance
        go.run(inst)
      })
      .catch(console.error)
  }
}

GoWasm.init()

```

### 使い方
`GoWasm.onLoad`が実行された後はこんな感じにgoの関数を実行できる。

```js
GoWasm.functions.echo({
  data: 'hello go',
  done: data => {
    console.log(data)
  },
  fail: console.error
})
```

### 補足
常に非同期な実行となるので注意。

wasmの仕様としては同期実行もできるっぽいが、`wasm_exec.js`で色々やっている都合なのか同期実行する方法にはたどり着けず。
