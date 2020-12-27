---
title: CTFlearn easy challenges
date: "2020-12-27T12:00:00.000Z"
template: "post"
draft: false
slug: "ctflearn-easy-challenges"
category: "Security"
tags:
  - "CTF"
  - "Security"
description: "About CTFlearn easy challenges; the commands and tools to solve these challenges"
socialImage: "/media/hacker.jpg"
---

![hacking](/media/hacker.jpg)

## What is CTFLearn
[CTFlearn](https://ctflearn.com) is an online platform to help hackers learn security by solving challenges.
There are many challenges including forensics, reverse engineering, cryptography, etc. Also, these challenges are categorised into easy, medium and hard levels.  
I have been playing on CTFLearn recently and solved more than 30 easy challenges. In this article, I introduce the commands and tools I've used to solve these challenges.

![CTFlearn](/media/ctf_learn.png)

## Easy challenges
Easy challenges are literally easy and simple. So these are good for security beginners like me.  
I have solved challenges related to cryptography, Forensics, programming, reverse engineering so far. Most challenges required me to download images, text or voice file. Then I found the flag from these files. This is what is called Forensics.

## Commands and tools to solve the challenges
Now, let's have a quick look at the commands and tools I've used.

- file

[file](https://en.wikipedia.org/wiki/File_(command)) command prints the format of files. Sometimes I got a file with an unclear format. In this case, by checking the format with the file command, I could treat the file properly.

```shell
└─[0] <> file Flag.txt 

Flag.txt: ASCII text, with no line terminators
```

- strings

[strings](https://en.wikipedia.org/wiki/Strings_(Unix)) command prints text embedded in binary or data file. Some challenges embedded the flag on a file. In this case, by extracting the text from the file, I could find the flag.
```shell
└─[0] <> strings RubberDuck.jpg

JFIF
CTFlearn{ILoveJakarta}
4ICC_PROFILE
$appl
mntrRGB XYZ
 acspAPPL
...
```

```shell
└─[0] <> strings RubberDuck.jpg | grep CTF

CTFlearn{ILoveJakarta}
```

- identify

[identify](https://imagemagick.org/script/identify.php) command prints the information of an image file. I could get important information with the identify command on some challenges.

```shell
└─[0] <> identify -verbose Pho.jpg

Image:
  Filename: Pho.jpg
  Format: JPEG (Joint Photographic Experts Group JFIF format)
  Mime type: image/jpeg
  Class: DirectClass
  Geometry: 595x661+0+0
  Units: Undefined
  Colorspace: sRGB
  Type: TrueColor
  Base type: Undefined
  Endianness: Undefined
...
```

- exiftool

[exiftool](https://exiftool.org/) prints the Exif information of an image file.

```shell
└─[0] <> exiftool Pho.jpg

ExifTool Version Number         : 12.11
File Name                       : Pho.jpg
Directory                       : .
File Size                       : 64 KiB
File Modification Date/Time     : 2020:12:20 18:45:35+01:00
File Access Date/Time           : 2020:12:26 16:30:27+01:00
File Inode Change Date/Time     : 2020:12:20 18:45:47+01:00
File Permissions                : rw-r--r--
File Type                       : JPEG
```

- dtmf

[dtmf-decoder](https://github.com/ribt/dtmf-decoder) extracts the phone number from an audio file.

```shell
└─[0] <> dtmf you_know_what_to_do.wav

67847010810197110123678289808479718265807289125
```

- Cypher Chef

In the above example, I could get the text by using dtmf-decorder. But what is the text format?  
The answer is the ASCII code. In this way, I frequently got an encryped or encoded flag on these challenges.
In order to get the flag, we finally have to decrypt or decode it. These challenges use some encryption and encoding algorithm such as ASCII, base64, Caesar cipher, XOR cipher, etc.

[CyberChef](https://gchq.github.io/CyberChef/) helps us in this situation. This platform provides so many ways for encryption, encoding, compression and data analysis.

![cyberchef](/media/cyberchef.png)

- Binwalk

Some challenges embed some files in a file.  
In this case, [Binwalk](https://github.com/ReFirmLabs/binwalk) helps us to extract the files from a file.

```shell
└─[0] <> binwalk --dd=".*" PurpleThing.png

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 780 x 720, 8-bit/color RGBA, non-interlaced
41            0x29            Zlib compressed data, best compression
153493        0x25795         PNG image, 802 x 118, 8-bit/color RGBA, non-interlaced
156776        0x26468         Unix path: /www.w3.org/1999/02/22-rdf-syntax-ns#">

# created the _PurpleThing.png.extracted directory including extracted files

└─[0] <> ls -a _PurpleThing.png.extracted
.  ..  0  25795  26468  29
```

- apktool

[apktool](https://ibotpeaches.github.io/Apktool/) is a reverse enginneering tool to decode an apk file.
```shell
└─[0] <> apktool d BasicAndroidRE1.apk

# created the BasicAndroidRE1 directory

└─[0] <> ls -a BasicAndroidRE1
.  ..  AndroidManifest.xml  apktool.yml  original  res  smali  unknown
```

- Programming

Some challenges required me to write code to solve, for example, the Luhn algorithm. I normally use Python.

## Conclusion
CTFlearn is super fun and very helpful to learn security. I really enjoy solving the challenge.  
My next step is medium challenges.  
In additional, [this site](https://trailofbits.github.io/ctf/) was helpful to learn the fundamental of CTF.
