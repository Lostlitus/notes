# Shell Note

## X11 forwarding

X11 forwarding makes it available for a remote server to display gui on local
monitor. It relies on X11 lib to work and is integrated into ssh. So we can
ssh to a remote server which has X11 lib installed, and run a X11 program to
see the result on the local monitor.

### Clipboard synchronization

Xclip(or xsel) is a X11 program that sends text to the local clipboard using
X11 forwarding, which means we can use it to synchronize clipboard between
local and remote.

First, we should specify the monitor for the remote to forward to. For example,
add this in the shell profile file.

```zsh
export DISPLAY="<local ip>:0.0"
```

Second, we just invoke it.

For example, in shell it can be

```zsh
echo 'text' | xclip
```

And in vim it can be

```vim
nnoremap <silent><leader>y :call system('xclip', @0)<CR>
```
