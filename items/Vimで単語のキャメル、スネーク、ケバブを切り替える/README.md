たまに欲しくなるけどさっと調べてもいい感じの方法が見つからず、そこまで必要に迫られてもなくて毎回手作業していた変換をやっとコマンドにしたメモ。
マップは空いていそうなキーへ適当に。

```vim
nnoremap <Space>c viw:s/\%V\(_\\|-\)\(.\)/\u\2/g<CR>
nnoremap <Space>_ viw:s/\%V\([A-Z]\)/_\l\1/g<CR>
nnoremap <Space>- viw:s/\%V\([A-Z]\)/-\l\1/g<CR>
xnoremap <Space>c :s/\%V\(_\\|-\)\(.\)/\u\2/g<CR>
xnoremap <Space>_ :s/\%V\([A-Z]\)/_\l\1/g<CR>
xnoremap <Space>- :s/\%V\([A-Z]\)/-\l\1/g<CR>
```

スネーク <-> ケバブの変換は使うシーンがなさそうなので略。

参考サイト
https://qiita.com/annyamonnya/items/f003fd671fecaf55c9e7
https://nanasi.jp/articles/howto/editing/replace-selecttext.html
