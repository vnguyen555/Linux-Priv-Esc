Dotfile code execution. Can be done with .bash_profile and
.bashrc
```
┌──(kali㉿kali)-[~]
└─$ echo "touch /tmp/bashtest.txt" >> ~/.bashrc
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~]
└─$ /bin/bash
┌──(kali㉿kali)-[~]
└─$ ls -al /tmp/bashtest.txt 
-rw-rw-r-- 1 kali kali 0 May 14 10:54 /tmp/bashtest.txt

┌──(kali㉿kali)-[~]
└─$ 


```