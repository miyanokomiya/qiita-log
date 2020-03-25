下記を参考にすると、
https://github.com/junegunn/fzf.vim/issues/276#issuecomment-268408009
https://github.com/junegunn/fzf/issues/337#issuecomment-136389913

↓でいけるようです。

```vim

" for Files
let $FZF_DEFAULT_COMMAND = 'ag --hidden --ignore .git -g ""'
" for Ag
autocmd VimEnter * command! -bang -nargs=* Ag
  \ call fzf#vim#ag(<q-args>, '--hidden --ignore .git', <bang>0)
```
