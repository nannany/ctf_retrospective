# fight

```
import random
import base64
import key

def xor(msg, key):
    return bytes([ch1^ch2 for ch1, ch2 in zip(msg, key)])

def gcd(x, y):
  while y != 0:
    r = x % y
    x = y
    y = r
  return x

def gen_seed(n):
  seed = 0
  for k in range(1,n):
    if gcd(k,n)==1:
      seed += 1
  return seed

s = 1
for p in b"Enjoy_HarekazeCTF!!":
  s *= p
seed = gen_seed(s)
random.seed(str(seed).rstrip("0"))

flag = key.FLAG
key = bytes([random.randint(0,255) for _ in flag])

enc = xor(flag, key)
print(base64.b64encode(enc).decode('utf-8')) #7XDZk9F4ZI5WpcFOfej3Dbau3yc1kxUgqmRCPMkzgyYFGjsRJF9aMaLHyDU=
```

いろいろ暗号化したものをBASE64エンコードしたら、
7XDZk9F4ZI5WpcFOfej3Dbau3yc1kxUgqmRCPMkzgyYFGjsRJF9aMaLHyDU=
になる。
そのひとつ前で `enc = xor(flag, key)`とあり、暗号化されたビット配列は、flagとkeyの排他的論理和であることが分かる。
そのため、既知である暗号化されたbit配列と、keyの排他的論理和を取れば、flagが分かる。

与えられたソースコードを少しいじって、keyを出そうと試みた。

```
import random
import base64

def xor(msg, key):
    return bytes([ch1^ch2 for ch1, ch2 in zip(msg, key)])

def gcd(x, y):
  while y != 0:
    r = x % y
    x = y
    y = r
  return x

def gen_seed(n):
  seed = 0
  for k in range(1,n):
    if gcd(k,n)==1:
      seed += 1
  return seed

s = 1
for p in b"Enjoy_HarekazeCTF!!":
  s *= p
# print(s)
seed = gen_seed(s)
random.seed(str(seed).rstrip("0"))
print(seed)
key = bytes([random.randint(0,255) for _ in range(45)])

print(key)
```

しかし、gen_seedに渡す引数が
4529255040439033800342855653030016000
と膨大であるため、for文を素直に回していたら間に合わない。

gen_seed でやっていることは、
1~4529255040439033800342855653030015999
のうち、
4529255040439033800342855653030016000
との最大公約数が1である数の個数を返している。

4529255040439033800342855653030016000
を素因数分解すると以下のよう
```
import sympy

print(sympy.factorint(4529255040439033800342855653030016000))
```
```
{2: 10, 3: 8, 5: 3, 7: 2, 11: 5, 19: 2, 23: 1, 37: 1, 53: 1, 61: 1, 67: 1, 97: 2, 101: 2, 107: 1}
```

包除原理を用いて、
4529255040439033800342855653030016000
と互いに素であるものの数を数える。
https://ja.wikipedia.org/wiki/%E5%8C%85%E9%99%A4%E5%8E%9F%E7%90%86

```
import itertools
import functools
import operator
import random
import base64

base64_ans = "7XDZk9F4ZI5WpcFOfej3Dbau3yc1kxUgqmRCPMkzgyYFGjsRJF9aMaLHyDU="
# print(base64.b64decode(base64_ans))

# 最後のBASE64化された文字列のビット配列
enc_flag_bit_array = ""
for tmp in base64.b64decode(base64_ans):
    enc_flag_bit_array += bin(tmp)[2:].zfill(8)

# 最終的な暗号化されたフラグのビット文字列
# print(enc_flag)

import sympy

# Sはランダム関数の種を生成する際に使う数
S = 4529255040439033800342855653030016000

factors = sympy.factorint(S)

# 包除原理を用いて解く
prime_factors_list = [p for p, _ in factors.items()]
# print(prime_factors_list)

# for tup in itertools.combinations(prime_factors_list,2):
#     print(tup)

# Sと互いに素にならない数の個数
not_tagainiso = 0

for i in range(1, len(prime_factors_list) + 1):
    for tup in itertools.combinations(prime_factors_list, i):
        if i & 1:
            not_tagainiso += S // functools.reduce(operator.mul, tup)
        else:
            not_tagainiso -= S // functools.reduce(operator.mul, tup)
seed = S - not_tagainiso
random.seed(str(seed).rstrip("0"))
key = bytes([random.randint(0, 255) for _ in range(45)])

key_bit_array = ""
for i in key:
    key_bit_array += bin(i)[2:].zfill(8)

ans_bit_array = ""
for e, k in zip(enc_flag_bit_array, key_bit_array):
    ans_bit_array += str(int(e) ^ int(k))

ans = ""
for j in range(0, len(enc_flag_bit_array), 8):
    ans += chr(int(ans_bit_array[j:j + 8], 2))

print(ans)
```

HarekazeCTF{3ul3rrrrrrrrr_t0000000t1nt!!!!!}

