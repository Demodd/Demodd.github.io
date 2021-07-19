



## [INSHack2017]10-cl0v3rf13ld-lane-signal

打开压缩包有一个.unk的文件，查看十六进制发现是jpg的图

![1620354114954](buumisc.assets/1620354114954.png)

其中还包含一张png的图片

![1620354231829](buumisc.assets/1620354231829.png)

在图片左下角，有一部分摩斯

![1620354327809](buumisc.assets/1620354327809.png)

```
.... . .-.. .--. -- .
```

在最后还有一部分ogg文件。

![1620354286234](buumisc.assets/1620354286234.png)

![1620354449589](buumisc.assets/1620354449589.png)

将其后缀改为.ogg后用audacity打开,又能得到一部分摩斯

![1620354526640](buumisc.assets/1620354526640.png)

```
.. -. ... .- -.--. -- ----- .-. ..... ...-- ..--.- .-- .---- .-.. .-.. ..--.- -. ...-- ...- ...-- .-. ..--.- ....- --. ...-- -.-.-- -.--.-
#flag{M0R53_W1LL_N3V3R_4G3!}
```

查看当前系统版本

```
volatility_2.6_win64_standalone.exe -f Twin.vmem imageinfo
```

## 蓝帽取证   I_will_but_not_quite 

![1620355776389](buumisc.assets/1620355776389.png)

```
volatility_2.6_win64_standalone.exe -f Twin.vmem --profile=Win7SP1x64 pslist
```

可以看到最后使用了winrar

![1620356070915](buumisc.assets/1620356070915.png)

![1620356772770](buumisc.assets/1620356772770.png)

```
volatility  -f Twin.vmem  --profile=Win7SP1x64 dumpfiles -Q 0x000000003e557990  -D / -u
```

![1620357649647](buumisc.assets/1620357649647.png)

```
outguess -k 123456 -r sea.jpg  flag.txt
```

```
4266gj2zn17b2jo5b62k73g22xg6j658350r5771vd40h4bd2ns33q30651y57s6752su3q05881hs3h53nb3603co2mv40l58n3da3f61i5
```

双十六进制编码,使用网站解密

```
https://www.calcresult.com/misc/cyphers/twin-hex.html
解得
Vnw3HC07BDgbBWNRGTx2fSckf399V1Z9CxIvHVd6fHsaEnR8fX40NyQ7JhM8CWV5fgMNN24=
```

根据一开始的加密脚本写解密脚本

加密脚本

```python
#!/user/bin/python2
import random
def r(s, num):
	l=""
	for i in s:
		if(ord(i) in range(97,97+26)):
			l+=chr((ord(i)-97+num)%26+97)
		else:
			l+=i
	return l

def x(a, b):
	return chr(ord(a)^ord(b))

def encrypt(c):
	secret = c
	n=random.randint(1,1000)
	for i in range(n):
		secret = r(secret, random.randint(1,26))
	secret = secret.encode('base64')

	l = ""
	for i in range(len(secret)):
		l += x(secret[i], secret[(i+1)%len(secret)])
	return l.encode('base64')

flag = "#################"
print "secret =", encrypt(flag)

#secret = The key you got
```



```python
import base64
import random
secret = "Vnw3HC07BDgbBWNRGTx2fSckf399V1Z9CxIvHVd6fHsaEnR8fX40NyQ7JhM8CWV5fgMNN24="
dec = base64.b64decode(secret).decode("utf-8")
# for i in range(len(dec)):
# 	print(ord(dec[i]))
def r(s, num): #凯撒
	l=""
	for i in s:
		if(ord(i) in range(97,97+26)):
			l+=chr((ord(i)-97+num)%26+97)
		else:
			l+=i
	return l

for i in range(86,128):
	j = 1
	tmp = [""]*len(dec)
	tmp[-1] = chr(i)#爆破恢复最后一位，即可恢复所有
	while j != len(dec):
		tmp[-j-1] = chr(ord(dec[-j])^ord(tmp[-j])) #反着进行异或
		j += 1
	s = tmp[-1] #因为最后一位是最后一位和第一位异或，所以刚开始异或的其实是最后一位
	for i in range(len(tmp)-1):
		s += tmp[i]#这里即是将第2位至最后一位拼接起来加在第一位后面
	try:
		s = base64.b64decode(s).decode("utf-8")
		for i in range(1,26):#遍历凯撒
			flag = r(s,i)
			print(flag)
	except:
		pass
```

![1620358114712](buumisc.assets/1620358114712.png)

```
flag{0946b1c23d7fe85afa951e7b2640dc83}
```

## [INSHack2018](not) so deep

打开wav附件,能看到一部分东西(能看到，但不能完全看到)

![1620379726333](buumisc.assets/1620379726333.png)

可以调一下频谱图设置，调一下高频

![1620379756209](buumisc.assets/1620379756209.png)

然后就能看到了,这样能得到一半flag

![1620379833734](buumisc.assets/1620379833734.png)

查看wp，另外一半flag被藏在了wav里，使用Deepsound提取

```python
#!/usr/bin/env python3
'''
deepsound2john extracts password hashes from audio files containing encrypted
data steganographically embedded by DeepSound (http://jpinsoft.net/deepsound/).
This method is known to work with files created by DeepSound 2.0.
Input files should be in .wav format. Hashes can be recovered from audio files
even after conversion from other formats, e.g.,
    ffmpeg -i input output.wav
Usage:
    python3 deepsound2john.py carrier.wav > hashes.txt
    john hashes.txt
This software is copyright (c) 2018 Ryan Govostes <rgovostes@gmail.com>, and
it is hereby released to the general public under the following terms:
Redistribution and use in source and binary forms, with or without
modification, are permitted.
'''

import logging
import os
import sys
import textwrap


def decode_data_low(buf):
  return buf[::2]

def decode_data_normal(buf):
  out = bytearray()
  for i in range(0, len(buf), 4):
    out.append((buf[i] & 15) << 4 | (buf[i + 2] & 15))
  return out

def decode_data_high(buf):
  out = bytearray()
  for i in range(0, len(buf), 8):
    out.append((buf[i] & 3) << 6     | (buf[i + 2] & 3) << 4 \
             | (buf[i + 4] & 3) << 2 | (buf[i + 6] & 3))
  return out


def is_magic(buf):
  # This is a more efficient way of testing for the `DSCF` magic header without
  # decoding the whole buffer
  return (buf[0] & 15)  == (68 >> 4) and (buf[2]  & 15) == (68 & 15) \
     and (buf[4] & 15)  == (83 >> 4) and (buf[6]  & 15) == (83 & 15) \
     and (buf[8] & 15)  == (67 >> 4) and (buf[10] & 15) == (67 & 15) \
     and (buf[12] & 15) == (70 >> 4) and (buf[14] & 15) == (70 & 15)


def is_wave(buf):
  return buf[0:4] == b'RIFF' and buf[8:12] == b'WAVE'


def process_deepsound_file(f):
  bname = os.path.basename(f.name)
  logger = logging.getLogger(bname)

  # Check if it's a .wav file
  buf = f.read(12)
  if not is_wave(buf):
    global convert_warn
    logger.error('file not in .wav format')
    convert_warn = True
    return
  f.seek(0, os.SEEK_SET)

  # Scan for the marker...
  hdrsz = 104
  hdr = None

  while True:
    off = f.tell()
    buf = f.read(hdrsz)
    if len(buf) < hdrsz: break

    if is_magic(buf):
          hdr = decode_data_normal(buf)
          logger.info('found DeepSound header at offset %i', off)
          break

    f.seek(-hdrsz + 1, os.SEEK_CUR)

  if hdr is None:
    logger.warn('does not appear to be a DeepSound file')
    return

  # Check some header fields
  mode = hdr[4]
  encrypted = hdr[5]

  modes = {2: 'low', 4: 'normal', 8: 'high'}
  if mode in modes:
    logger.info('data is encoded in %s-quality mode', modes[mode])
  else:
    logger.error('unexpected data encoding mode %i', modes[mode])
    return

  if encrypted == 0:
    logger.warn('file is not encrypted')
    return
  elif encrypted != 1:
    logger.error('unexpected encryption flag %i', encrypted)
    return

  sha1 = hdr[6:6+20]
  print('%s:$dynamic_1529$%s' % (bname, sha1.hex()))


if __name__ == '__main__':
  import argparse

  parser = argparse.ArgumentParser()
  parser.add_argument('--verbose', '-v', action='store_true')
  parser.add_argument('files', nargs='+', metavar='file',
    type=argparse.FileType('rb', bufsize=4096))
  args = parser.parse_args()

  if args.verbose:
    logging.basicConfig(level=logging.INFO)
  else:
    logging.basicConfig(level=logging.WARN)

  convert_warn = False

  for f in args.files:
    process_deepsound_file(f)

  if convert_warn:
    print(textwrap.dedent('''
    ---------------------------------------------------------------
    Some files were not in .wav format. Try converting them to .wav
    and try again. You can use: ffmpeg -i input output.wav
    ---------------------------------------------------------------
    '''.rstrip()), file=sys.stderr)
    
    
```

使用此脚本获取文件的hash值,然后使用john进行破解

![1620384592383](buumisc.assets/1620384592383.png)

得到密码解密获得另一半flag

![1620384610921](buumisc.assets/1620384610921.png)

```
flag{Aud1o_st3G4n0_1s_4lwayS_Th3_S4me}
```

## 蓝帽 冬奥会_is_coming

![1620385443382](buumisc.assets/1620385443382.png)





```
zsteg -a 'b8,rgb,lsb,xy' attachment\(1\).png 
```

![1620391885528](buumisc.assets/1620391885528.png)

## ctfshow misc49

字母解密:https://www.qqxiuzi.cn/bianma/wenbenjiami.php?s=zimu

音符解密->花朵解密->解出压缩包的key

```
python2 bwm.py decode 1.png 2.png 3.png
```

![1620462528950](buumisc.assets/1620462528950.png)

```
flag{y0u_are_we1l}
```

## misc6

字母解密

![1620463238926](buumisc.assets/1620463238926.png)



## misc8

明文攻击

![1620463829492](buumisc.assets/1620463829492.png)

扫码拿到flag

## stega2

改高度

![1620464176674](buumisc.assets/1620464176674.png)

```
flag{na yi nian wo ye bian cheng le guang}
```

## stega3

图片结尾提示NTFS

![1620464417198](buumisc.assets/1620464417198.png)

```
flag{ntfs_is_so_cool}
```

## stega4_STEGA 	

![1620464684878](buumisc.assets/1620464684878.png)

## 文本隐写 	

docx->zip->word->document.xml->

```
RmxhZyU3QnNob3dfY3RmX3Rzd19jYyU3RA==
```

```
Flag{show_ctf_tsw_cc}
```

## misc21

猜猜看---->>> outguess

```
outguess -k 123456 -r 1.jpg hid.txt
```

010查看十六进制发现hid.txt是一个zip，在zip后还隐藏了一些东西

![1620465661579](buumisc.assets/1620465661579.png)



```
tab 替换 -
空格 替换 .
tab 替换/
\n 替换/
```

![1620465690954](buumisc.assets/1620465690954.png)

得到压缩包密码，解码拿到flag

```
flag{023032132-asdasdasd-ljklkz}
```

## stega12

png动图(ps:忘了叫啥了)，拖到浏览器即可

```
flag{letmeseesee}
```

## stega5

stegsolve查看发现隐藏了二维码

![1624610443877](buumisc.assets/1624610443877.png)

扫码得到

```
03F30D0A79CB05586300000000000000000100000040000000730D0000006400008400005A000064010053280200000063000000000300000016000000430000007378000000640100640200640300640400640500640600640700640300640800640900640A00640600640B00640A00640700640800640C00640C00640D00640E00640900640F006716007D00006410007D0100781E007C0000445D16007D02007C01007400007C0200830100377D0100715500577C010047486400005328110000004E6966000000696C00000069610000006967000000697B000000693300000069380000006935000000693700000069300000006932000000693400000069310000006965000000697D000000740000000028010000007403000000636872280300000074030000007374727404000000666C6167740100000069280000000028000000007304000000312E7079520300000001000000730A0000000001480106010D0114014E280100000052030000002800000000280000000028000000007304000000312E707974080000003C6D6F64756C653E010000007300000000
```

发现为pyc的十六进制文件，反编译一下(pyc文件可以考虑pyc反编译和pyc隐写)

```
uncompyle6 -o . 1.pyc
```

```python

str = [
     102, 108, 97, 103, 123, 51, 56, 97, 53, 55, 48, 51, 50, 48, 56, 53, 52, 52, 49, 101, 55, 125]
flag = ''
for i in str:
    flag += chr(i)

print(flag)
```

```
flag{38a57032085441e7}
```

## misc7

经过谷歌

```
D0 CF 11 E0 A1 B1 1A E1为office的文件头
```

将其改为docx发现有密码，使用工具对其进行破解

![1624625075606](buumisc.assets/1624625075606.png)

得到密码,解密发现文件格式不对，将其改为ppt正确打开，搜索flag，并将其改为可见的颜色

![1624625145632](buumisc.assets/1624625145632.png)

## [羊城杯 2020]TCP_IP

查看了每一个包的data部分没发现什么东西，但是在identification发现了每个包的ascii码值不同，可能为本题的考点，尝试将其提取出来

![1624531109819](buumisc.assets/1624531109819.png)

```
 tshark -r attachment(1).pcap -T fields -e ip.id
```

![1624531190719](buumisc.assets/1624531190719.png)

python转换一下

```python
s = '''0x00000040
0x00000069
0x00000048
0x0000003c
0x0000002c
0x0000007b
0x0000002a
0x0000003b
0x0000006f
0x00000055
0x00000070
0x0000002f
0x00000069
0x0000006d
0x00000022
0x00000051
0x00000050
0x0000006c
0x00000060
0x00000079
0x00000052
0x0000002a
0x00000069
0x00000065
0x0000007d
0x0000004e
0x0000004b
0x0000003b
0x0000002e
0x00000044
0x00000021
0x00000058
0x00000075
0x00000029
0x00000062
0x0000003a
0x0000004a
0x0000005b
0x00000052
0x0000006a
0x0000002b
0x00000036
0x0000004b
0x0000004b
0x0000004d
0x00000037
0x00000050
0x00000040
0x00000069
0x00000048
0x0000003c
0x0000002c
0x0000007b
0x0000002a
0x0000003b
0x0000006f
0x00000055
0x00000070
0x0000002f
0x00000069
0x0000006d
0x00000022
0x00000051
0x00000050
0x0000006c
0x00000060
0x00000079
0x00000052'''
a = s.split('\n')
# print(a)
list = []
for i in a:
    list.append(i[-2:])
# print(list)
reasult = ''
for i in list:
    reasult += chr(int(i,16))

print(reasult)
```

```
@iH<,{*;oUp/im"QPl`yR*ie}NK;.D!Xu)b:J[Rj+6KKM7P@iH<,{*;oUp/im"QPl`yR

@iH<,{*;oUp/im"QPl`yR*ie}NK;.D!Xu)b:J[Rj+6KKM7P
```

多了一部分数据

```
flag{wMt84iS06mCbbfuOfuVXCZ8MSsAFN1GA}
```





​				