# Rule86
解けず。
Cryptの問題。75人が正解。
問題文は以下。
> Kevin is working on a new synchronous stream cipher, but he has been re-using his key.

グーグル翻訳
> Kevinは新しい同期ストリーム暗号に取り組んでいますが、彼は自分の鍵を再利用しています。

問題文中にリンクが張られていて、
3つの暗号ファイル
* hint.gif.enc　(24252byte)
* super_cipher.py.enc (768byte)
* rule86.txt.enc (512byte)

1つの平文
* rule86.txt (511byte)

がダウンロードできる。

rule86.txt の中身は、

```
What is Rule 86?

Rule 86 DOES NOT state that:
- any and all buttons, valves, switches, latches, or anything else may be pushed or turned more than once until one is certain that all buttons, valves, switches, latches, etc, has been pushed at least once each
- every time someone speaks your name, it creates a duplicate of you
- if one divides 86 by the sugar content of sap, you can estimate the amount of sap required to produce a gallon of syrup
- sharing a kiss is forbidden

Rule 86 is... something else.
```

これをストリーム暗号で暗号化したものが `rule86.txt.enc` 。

ストリーム暗号についての説明は[こちら](https://www.cyphertec.co.jp/techno/t_stream.html)。
>ストリーム暗号は、暗号鍵をシード(乱数の生成源となる値)として鍵ストリームと呼ばれる疑似乱数を生成し、平文との排他的論理和（XOR）を求めることで暗号化を行います。

とある。

`rule86.txt` は `rule86.txt.emc` の平文であるので、これらの排他的論理和をとれば、rule86.txtを暗号化した鍵ストリームは明らかになる。
また、`彼は自分の鍵を再利用しています。` とあることから、他の2つの暗号文も同じ鍵ストリームで暗号化されていることが予想される。

以下のスクリプトで `super_cipher.py.enc` を一部平文に戻した。
```
def main():
    encryped_data = open('rule86.txt.enc', 'rb').read()
    decryped_data = open('rule86.txt', 'rb').read()

    global key
    key = []
    for enc, dec in zip(encryped_data, decryped_data):
        key.append(enc ^ dec)

    e_super_cipher = open('super_cipher.py.enc', 'rb').read()

    d_super_cipher = []
    for sc, k in zip(e_super_cipher, key):
        d_super_cipher.append(sc ^ k)

    tmp = list(map(chr, d_super_cipher))
    print("".join(tmp))

if __name__ == '__main__':
    main()
```

平文の一部
```
#!/usr/bin/env python3

import argparse
import sys

parser = argparse.ArgumentParser()
parser.add_argument("key")
args = parser.parse_args()

RULE = [86 >> i & 1 for i in range(8)]
N_BYTES = 32
N = 8 * N_BYTES

def next(x):
  x = (x & 1) << N+1 | x << 1 | x >> N-1
  y = 0
  for i in range(N):
    y |= RULE[(x >> i) & 7] << i
  return y

# Bootstrap the PNRG
keystream = int.from_bytes(args.key.encode(),'little')
for i in range(N//2):
  keystream = next(keystream)

# Encrypt / decrypt stdin to stdout
plainte
```

この平文の一部を読み解いていく。
まず、`RULE = [86 >> i & 1 for i in range(8)]`
の部分は計算すると、`RULE = [0, 1, 1, 0, 1, 0, 1, 0]` になるのが分かる。
(>> は左シフト、 &は論理積、|は論理和 詳しくは[こちら](http://www.tohoho-web.com/python/operators.html))





