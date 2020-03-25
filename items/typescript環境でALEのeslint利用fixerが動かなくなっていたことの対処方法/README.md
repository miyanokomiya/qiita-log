## 困った
手元で`eslint --fix`すると動くのに、ALE経由だと何も起きなくなってしまった。

原因は最近出るようになったtypescriptバージョン関連の警告。
ALEは標準出力をパースしていい感じにしてくれているが、この警告が邪魔をしてパースに失敗していた。
パースしてても正常系で通ってしまうらしく、エラー表示も何もなかったので見つけるのになかなか手間取った。

```sh
$ cat src/components/Shelf.tsx| npx eslint --parser-options=warnOnUnsupportedTypeScriptVersion:true  --stdin-filename src/components/Shelf.tsx --stdin --fix-dry-run --format=json
=============

WARNING: You are currently running a version of TypeScript which is not officially supported by typescript-estree.

You may find that it works just fine, or you may not.

SUPPORTED TYPESCRIPT VERSIONS: >=3.2.1 <3.6.0

YOUR TYPESCRIPT VERSION: 3.6.2

Please only submit bug reports when using the officially supported version.

=============
[{"filePath":"~/develop/web/rn-sample/src/components/Shelf.tsx","messages":[],"errorCount":0,"warningCount":0,"fixableErrorCount":0,"fixableWarningCount":0,"output":"import React, { FC } from 'react'\n\nconst Shelf: FC = () => {\n  return <div>aaaa</div>\n}\n\nexport default Shelf\n"}]
```

とても存在感のある警告表示。しかしそこは後続連携で大事な大事な標準出力なのである。

## 対処法
https://github.com/typescript-eslint/typescript-eslint/issues/393#issuecomment-511115374
オプションで警告を消すことができるので消してしまう。
あるいは警告に素直に従ってバージョンを調整する。

ALEへのPRチャンスかと一瞬思ったが、eslint本体ではなくtypescript用のプラグインが警告出力を上乗せしてきているっぽいので、ALE側で何とかするべきものでもなさそうな気配。
せめて他の嵌り人の助けになってくれということでここにメモ。
