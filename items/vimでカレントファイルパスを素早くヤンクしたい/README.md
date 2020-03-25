`<C-g>`で表示はすぐできるのに、ヤンクはすぐできないもどかしさを解消したかった。

```vim
function! ClipText(data)
  let @0=a:data
  let @"=a:data
  let @*=a:data
endfunction
nnoremap <C-g> :call ClipText(expand('%'))<CR><C-g>
```

Thanks: https://gist.github.com/pinzolo/8168337
