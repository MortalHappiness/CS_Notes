### Background:

The plugin `Ultisnip` in vim throws the following error when enter insert mode and type any text.

### Error:

```
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/home/mortalhappiness/miniconda3/lib/python3.7/subprocess.py", line 152, in <module>
    import _posixsubprocess
ImportError: /home/mortalhappiness/miniconda3/lib/python3.7/lib-dynload/_posixsubprocess.cpython-37m-x86_64-linux-gnu.so: undefined symbol: _Py_write_noraise
```

### Reproduce:

Type the following command in vim
```
:py3 import subprocess
```

### Solution:

Replace miniconda path in PATH.
Add the following line into `.bashrc`
```
alias vim='PATH=`echo $PATH | sed "s/\/home\/mortalhappiness\/miniconda3\/bin://"` vim'
```

### Reference:

1. https://blog.kali-team.cn/2019/04/14/Arch%E5%90%84%E7%A7%8D%E9%97%AE%E9%A2%98%E8%A7%A3%E5%86%B3%E6%96%B9%E6%B3%95/
2. https://www.reddit.com/r/vim/comments/8r2oym/vim_help_vim_cannot_import_subprocess_python3/e0o0ivr/
