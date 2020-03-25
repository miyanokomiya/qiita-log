## 問題意識
[fzf](https://github.com/junegunn/fzf)により生産性とQuality of Vimが桁違いに向上している今日このごろ、`:Ag`コマンドで隠しファイル（ドットファイル、dotfiles）が検索対象ファイルにならないことが密かに気になっていた。

そしてやっと隠しファイルも検索対象とする設定を書けたので、もしかしたら世界の誰かのQOV向上に貢献できるかもしれないということでここにメモ。  

## 解決方法
プラグインは[deiin](https://github.com/Shougo/dein.vim)で管理しているので`.dein.toml`形式で抜粋。fzfと[ag](https://github.com/ggreer/the_silver_searcher)は別途インストール済ということで。

```vim
# .dein.toml

[[plugins]]
repo = 'junegunn/fzf.vim'
hook_add = '''
" fzfは単体でインストールしているのでそこへのパス
set rtp+=/usr/local/opt/fzf
" dotファイルを含める&プレビュー
command! -bang -nargs=? -complete=dir Files
  \ call fzf#vim#files(<q-args>, fzf#vim#with_preview({'source': 'ag --hidden --ignore .git -g ""'}), <bang>0)
command! -bang -nargs=* Ag
  \ call fzf#vim#grep(
  \   'ag --column --color --hidden --ignore .git '.shellescape(<q-args>), 0,
  \   <bang>0 ? fzf#vim#with_preview('up:60%')
  \           : fzf#vim#with_preview('right:50%', '?'),
  \   <bang>0)
'''
```

[fzf.vimのREADME](https://github.com/junegunn/fzf.vim#advanced-customization)を参考に、`:Files`コマンドは`.gitignore`されていない全ファイルを対象にする。この書き方のソースは失念、多分検索すれば似たようなものがでてくるはず。

そして今回やっと成功したのが`:Ag`コマンドの設定。READMEに書いてある`fzf#vim#ag`を`call`する書き方だと`ag`への引数は[内部で固定](https://github.com/junegunn/fzf.vim/blob/7bf940d261795b5164bc854f37a121fcca927941/autoload/fzf/vim.vim#L691-L696)されてしまっているらしく、うまくいかなかった。
なのでこちらは諦めて、`:Rg`用の書き方として載っていた`fzf#vim#grep`を`call`する書き方を弄ってみたらうまくいった。

第二引数に`0`を渡しているのは、`1`を渡したら`:Ag`で検索したファイルを開いたときになぜか同時に文字列（検索ワード？）が書き込まれることがあるという謎の現象に遭遇したため。`0`に変えてみたら再現しなくなったのでこれで落ち着いている。
[fzfに`column`オプションを指定](https://github.com/junegunn/fzf.vim/blob/7bf940d261795b5164bc854f37a121fcca927941/autoload/fzf/vim.vim#L710-L716)するための引数っぽいが、よく分からない。`ag --column`と同じ機能で、fzf側でも指定すると競合して変な挙動になるとかかもしれない。求むfzf強者。
