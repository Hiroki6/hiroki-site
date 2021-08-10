---
title: CTFlearn Writeups - Finish the flag -
date: "2021-08-10T12:00:00.000Z"
template: "post"
draft: false
slug: "writesup-finish-the-flag"
category: "Reverse Engineering"
tags:
  - "CTF"
  - "Security"
  - "Reverse Engineering"
description: "Writeups of 'Finish the flag' in CTFlearn reverse engineering challenge"
socialImage: "/media/letter.png"
---

![hacking](/media/letter.png)

This is a writeups of [Finish the flag](https://ctflearn.com/challenge/1122) in [CTFlearn](https://ctflearn.com). This question is categoried in `Reverse Engineering` and the difficulty is Easy.


>> I received a strange letter in the mail, when I unfolded the document inside, I discovered this matrix bar code. Can you figure out what it contains?

I downloaded the `letter.zip` file and unzip it. It includes these files. The `qr.png` file includes the flag I have to get. The content of this file is a QR code.
```
└─[0] <> ls
qr.asm.enc  qr.png  readme.txt
```

I scanned the QR code in the png file. It looks a base64 format string.
```
f0VMRgEBAQAAAAAAAAAAAAIAAwABAAAAgIAECDQAAACMAgAAAAAAADQAIAACACgABgAFAAEAAAAAAAAAAIAECACABAjFAAAAxQAAAAUAAAAAEAAAAQAAAMgAAADIkAQIyJAECFcAAABXAAAABgAAAAAQAAAAAAAAAAAAAAAAAAC6CgAAALkUkQQIuwEAAAC4BAAAAM2AMcA8B3Qdi5DkkAQIQOkAAAAAVYnlUIDyF4hVAFhd6d////+7AAAAALgBAAAAzYAAAAA5w6nhu7PDvHtqbj09fSAgPFw6ICBfXyAgUDAARkVIYSQnagBRw7NBOGTDunAwbTk5cse5MDQwVjA1ZmYxTGxrOVwwb09cQy9cMDAAQ1RGbGVhcm57CgAAAAAAAAAAAAAAAAAAAAAAAAAAAACAgAQIAAAAAAMAAQAAAAAAyJAECAAAAAADAAIAAQAAAAAAAAAAAAAABADx/wgAAADIkAQIAAAAAAAAAgAPAAAA5JAECAAAAAAAAAIAFQAAAOyQBAgAAAAAAAACAB0AAAAUkQQIAAAAAAAAAgAjAAAAmIAECAAAAAAAAAEAKAAAAKiABAgAAAAAAAABADUAAAC5gAQIAAAAAAAAAQA/AAAAgIAECAAAAAAQAAEAOgAAAB+RBAgAAAAAEAACAEYAAAAfkQQIAAAAABAAAgBNAAAAIJEECAAAAAAQAAIAAHFyLmFzbQByYW5kb20AZWZsYWcAcmFuZG9tMgBzZmxhZwBsb29wAGJpdGVfb2ZfZmxhZwBkb25lAF9fYnNzX3N0YXJ0AF9lZGF0YQBfZW5kAAAuc3ltdGFiAC5zdHJ0YWIALnNoc3RydGFiAC50ZXh0AC5kYXRhAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAbAAAAAQAAAAYAAACAgAQIgAAAAEUAAAAAAAAAAAAAABAAAAAAAAAAIQAAAAEAAAADAAAAyJAECMgAAABXAAAAAAAAAAAAAAAEAAAAAAAAAAEAAAACAAAAAAAAAAAAAAAgAQAA8AAAAAQAAAALAAAABAAAABAAAAAJAAAAAwAAAAAAAAAAAAAAEAIAAFIAAAAAAAAAAAAAAAEAAAAAAAAAEQAAAAMAAAAAAAAAAAAAAGICAAAnAAAAAAAAAAAAAAABAAAAAAAAAA==
```

By decoding the base64 code, I got the binary below. It looks an ELF.
```python
In [1]: import base64

In [2]: data = 'f0VMRgEBAQAAAAAAAAAAAAIAAwABAAAAgIAECDQAAACMAgAAAAAAADQAIAACACgABgAFAAEAAAAAAAAAAIAECAC
   ...: ABAjFAAAAxQAAAAUAAAAAEAAAAQAAAMgAAADIkAQIyJAECFcAAABXAAAABgAAAAAQAAAAAAAAAAAAAAAAAAC6CgAAALkUkQ
   ...: QIuwEAAAC4BAAAAM2AMcA8B3Qdi5DkkAQIQOkAAAAAVYnlUIDyF4hVAFhd6d////+7AAAAALgBAAAAzYAAAAA5w6nhu7PDv
   ...: Htqbj09fSAgPFw6ICBfXyAgUDAARkVIYSQnagBRw7NBOGTDunAwbTk5cse5MDQwVjA1ZmYxTGxrOVwwb09cQy9cMDAAQ1RG
   ...: bGVhcm57CgAAAAAAAAAAAAAAAAAAAAAAAAAAAACAgAQIAAAAAAMAAQAAAAAAyJAECAAAAAADAAIAAQAAAAAAAAAAAAAABAD
   ...: x/wgAAADIkAQIAAAAAAAAAgAPAAAA5JAECAAAAAAAAAIAFQAAAOyQBAgAAAAAAAACAB0AAAAUkQQIAAAAAAAAAgAjAAAAmI
   ...: AECAAAAAAAAAEAKAAAAKiABAgAAAAAAAABADUAAAC5gAQIAAAAAAAAAQA/AAAAgIAECAAAAAAQAAEAOgAAAB+RBAgAAAAAE
   ...: AACAEYAAAAfkQQIAAAAABAAAgBNAAAAIJEECAAAAAAQAAIAAHFyLmFzbQByYW5kb20AZWZsYWcAcmFuZG9tMgBzZmxhZwBs
   ...: b29wAGJpdGVfb2ZfZmxhZwBkb25lAF9fYnNzX3N0YXJ0AF9lZGF0YQBfZW5kAAAuc3ltdGFiAC5zdHJ0YWIALnNoc3RydGF
   ...: iAC50ZXh0AC5kYXRhAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAbAAAAAQAAAAYAAACAgA
   ...: QIgAAAAEUAAAAAAAAAAAAAABAAAAAAAAAAIQAAAAEAAAADAAAAyJAECMgAAABXAAAAAAAAAAAAAAAEAAAAAAAAAAEAAAACA
   ...: AAAAAAAAAAAAAAgAQAA8AAAAAQAAAALAAAABAAAABAAAAAJAAAAAwAAAAAAAAAAAAAAEAIAAFIAAAAAAAAAAAAAAAEAAAAA
   ...: AAAAEQAAAAMAAAAAAAAAAAAAAGICAAAnAAAAAAAAAAAAAAABAAAAAAAAAA=='

In [3]: encodedBytes = base64.b64decode(data)

In [4]: print(encodedBytes)
b"\x7fELF\x01\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\x03\x00\x01\x00\x00\x00\x80\x80\x04\x084\x00\x00\x00\x8c\x02\x00\x00\x00\x00\x00\x004\x00 \x00\x02\x00(\x00\x06\x00\x05\x00\x01\x00\x00\x00\x00\x00\x00\x00\x00\x80\x04\x08\x00\x80\x04\x08\xc5\x00\x00\x00\xc5\x00\x00\x00\x05\x00\x00\x00\x00\x10\x00\x00\x01\x00\x00\x00\xc8\x00\x00\x00\xc8\x90\x04\x08\xc8\x90\x04\x08W\x00\x00\x00W\x00\x00\x00\x06\x00\x00\x00\x00\x10\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xba\n\x00\x00\x00\xb9\x14\x91\x04\x08\xbb\x01\x00\x00\x00\xb8\x04\x00\x00\x00\xcd\x801\xc0<\x07t\x1d\x8b\x90\xe4\x90\x04\x08@\xe9\x00\x00\x00\x00U\x89\xe5P\x80\xf2\x17\x88U\x00X]\xe9\xdf\xff\xff\xff\xbb\x00\x00\x00\x00\xb8\x01\x00\x00\x00\xcd\x80\x00\x00\x009\xc3\xa9\xe1\xbb\xb3\xc3\xbc{jn==}  <\\:  __  P0\x00FEHa$'j\x00Q\xc3\xb3A8d\xc3\xbap0m99r\xc7\xb9040V05ff1Llk9\\0oO\\C/\\00\x00CTFlearn{\n\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x80\x80\x04\x08\x00\x00\x00\x00\x03\x00\x01\x00\x00\x00\x00\x00\xc8\x90\x04\x08\x00\x00\x00\x00\x03\x00\x02\x00\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x04\x00\xf1\xff\x08\x00\x00\x00\xc8\x90\x04\x08\x00\x00\x00\x00\x00\x00\x02\x00\x0f\x00\x00\x00\xe4\x90\x04\x08\x00\x00\x00\x00\x00\x00\x02\x00\x15\x00\x00\x00\xec\x90\x04\x08\x00\x00\x00\x00\x00\x00\x02\x00\x1d\x00\x00\x00\x14\x91\x04\x08\x00\x00\x00\x00\x00\x00\x02\x00#\x00\x00\x00\x98\x80\x04\x08\x00\x00\x00\x00\x00\x00\x01\x00(\x00\x00\x00\xa8\x80\x04\x08\x00\x00\x00\x00\x00\x00\x01\x005\x00\x00\x00\xb9\x80\x04\x08\x00\x00\x00\x00\x00\x00\x01\x00?\x00\x00\x00\x80\x80\x04\x08\x00\x00\x00\x00\x10\x00\x01\x00:\x00\x00\x00\x1f\x91\x04\x08\x00\x00\x00\x00\x10\x00\x02\x00F\x00\x00\x00\x1f\x91\x04\x08\x00\x00\x00\x00\x10\x00\x02\x00M\x00\x00\x00 \x91\x04\x08\x00\x00\x00\x00\x10\x00\x02\x00\x00qr.asm\x00random\x00eflag\x00random2\x00sflag\x00loop\x00bite_of_flag\x00done\x00__bss_start\x00_edata\x00_end\x00\x00.symtab\x00.strtab\x00.shstrtab\x00.text\x00.data\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x1b\x00\x00\x00\x01\x00\x00\x00\x06\x00\x00\x00\x80\x80\x04\x08\x80\x00\x00\x00E\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x10\x00\x00\x00\x00\x00\x00\x00!\x00\x00\x00\x01\x00\x00\x00\x03\x00\x00\x00\xc8\x90\x04\x08\xc8\x00\x00\x00W\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x04\x00\x00\x00\x00\x00\x00\x00\x01\x00\x00\x00\x02\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00 \x01\x00\x00\xf0\x00\x00\x00\x04\x00\x00\x00\x0b\x00\x00\x00\x04\x00\x00\x00\x10\x00\x00\x00\t\x00\x00\x00\x03\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x10\x02\x00\x00R\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x01\x00\x00\x00\x00\x00\x00\x00\x11\x00\x00\x00\x03\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00b\x02\x00\x00'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x01\x00\x00\x00\x00\x00\x00\x00"
```

I wrote the binary content into a file and executed it.
```python
execute_file = open("a.out", "wb")
execute_file.write(encodedBytes)
```

I got just `CTFlearn{`, but confirmed this file is executable.
```
└─[0] <> ./a.out
CTFlearn{
```

This is the resulf of debugging the execute file into assembly format. It looks very simple and seems there is a loop.
```
└─[0] <> objdump -S -M intel a.out

a.out:     file format elf32-i386


Disassembly of section .text:

08048080 <_start>:
 8048080:       ba 0a 00 00 00          mov    edx,0xa
 8048085:       b9 14 91 04 08          mov    ecx,0x8049114
 804808a:       bb 01 00 00 00          mov    ebx,0x1
 804808f:       b8 04 00 00 00          mov    eax,0x4
 8048094:       cd 80                   int    0x80
 8048096:       31 c0                   xor    eax,eax

08048098 <loop>:
 8048098:       3c 07                   cmp    al,0x7
 804809a:       74 1d                   je     80480b9 <done>
 804809c:       8b 90 e4 90 04 08       mov    edx,DWORD PTR [eax+0x80490e4]
 80480a2:       40                      inc    eax
 80480a3:       e9 00 00 00 00          jmp    80480a8 <bite_of_flag>

080480a8 <bite_of_flag>:
 80480a8:       55                      push   ebp
 80480a9:       89 e5                   mov    ebp,esp
 80480ab:       50                      push   eax
 80480ac:       80 f2 17                xor    dl,0x17
 80480af:       88 55 00                mov    BYTE PTR [ebp+0x0],dl
 80480b2:       58                      pop    eax
 80480b3:       5d                      pop    ebp
 80480b4:       e9 df ff ff ff          jmp    8048098 <loop>

080480b9 <done>:
 80480b9:       bb 00 00 00 00          mov    ebx,0x0
 80480be:       b8 01 00 00 00          mov    eax,0x1
 80480c3:       cd 80                   int    0x80
```

This loop is executed up to 7times.
```
08048098 <loop>:
 8048098:       3c 07                   cmp    al,0x7
 804809a:       74 1d                   je     80480b9 <done>
```

In the `bite_of_flag` function, a character is defined. I assumed this character is the content of the flag.
```
80480ac:       80 f2 17                xor    dl,0x17
80480af:       88 55 00                mov    BYTE PTR [ebp+0x0],dl
```

By debugging with `gdb`, I found the characters have hex format. 
```
gdb-peda$ i registers dl
dl             0x51                0x51
```

```python
In [1]: chr(0x51)
Out [1]: 'Q'
```

By debugging the loop, I can get the all 7 characters, but it's bit combersom. Therefore, I wrote the simple script below to get all characters.
```python
import gdb

gdb.execute('break *0x80480af')
gdb.execute('run')

flag = ''
for i in range(7):
    dl = gdb.parse_and_eval('$dl')
    flag += chr(dl)

    gdb.execute('continue')

print("CTFlearn{" + flag)
```

By executing the command below, I got the flag.
```
└─[0] <> gdb a.out -x ./exploit.py
CTFlearn{QR_v30}
```

As it's written in the `readme.txt`, you can get the original assmembly file by this command below.
```
openssl enc -d -aes-256-cbc -pbkdf2 -k CTFlearn{QR_v30} -in qr.asm.enc -out qr.asm
```