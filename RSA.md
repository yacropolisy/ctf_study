# RSA

## 基本

公開鍵：n, e

秘密鍵：d (p, q) 

n = p * q で、p, q がわかればdは分かるので、n を素因数分解できれば勝ち。

暗号文cから平文mを復号 →  $m = c^d \mod n $  

## p, q を手に入れてからの復号

```python
import gmpy

def decrypt(p, q, e, c):
    l = gmpy.lcm(p-1, q-1)
    gcd, u, v = gmpy.gcdext(e, l)
    if u < 0:
        u += l
    n = p * q
    d = u
    m = pow(c, d, n)
```



## 10進数→文字列

```python
import codecs
print(codecs.decode(('%x'%m),'hex_codec'))
```

```python
from Crypto.Util.number import long_to_bytes
print(long_to_bytes(m))
```



## RSA-CRT

CRT(中国剰余定理)を用いたRSA

秘密鍵として、DP、DQ、QINVが使われる。

QINVは $q^{-1}\mod p$

```python
import gmpy2
from Crypto.Util.number import long_to_bytes

def decrypt(c, p, q, dp, dq):
    qinv = gmpy2.invert(q, p)
    m1 = pow(c, dp, p)
    m2 = pow(c, dq, q)
    h = (qinv * (m1 - m2)) % p
    m = m2 + h * q
    return long_to_bytes(m)
```



## eが小さい時

eが小さいとき、nのe乗根以下の平文mについては、単純に暗号文cのe乗根を取れば求めることができる。

```python
import sys
import gmpy
import codecs

def root_e(c, e, n):
    bound = gmpy.root(n, e)[0]
    m = gmpy.root(c, e)[0]
    return m, bound


if __name__ == '__main__':
    n = 374159235470172130988938196520880526947952521620932362050308663243595788308583992120881359365258949723819911758198013202644666489247987314025169670926273213367237020188587742716017314320191350666762541039238241984934473188656610615918474673963331992408750047451253205158436452814354564283003696666945950908549197175404580533132142111356931324330631843602412540295482841975783884766801266552337129105407869020730226041538750535628619717708838029286366761470986056335230171148734027536820544543251801093230809186222940806718221638845816521738601843083746103374974120575519418797642878012234163709518203946599836959811
    e = 3

    c = 2205316413931134031046440767620541984801091216351222789180593875373829950860542792110364325728088504479780803714561464250589795961097670884274813261496112882580892020487261058118157619586156815531561455215290361274334977137261636930849125

    m, bound = root_e(c, e, n)
    print(codecs.decode(('%x'%m),'hex_codec'))

```



## eが大きい時

eが極端に大きければ、wiener-attackというのが使えるらしい。

```python
import sys
import gmpy
import codecs

def continued_fraction(n, d):
    """
    415/93 = 4 + 1/(2 + 1/(6 + 1/7))

    >>> continued_fraction(415, 93)
    [4, 2, 6, 7]
    """
    cf = []
    while d:
        q = n // d
        cf.append(q)
        n, d = d, n-d*q
    return cf

def convergents_of_contfrac(cf):
    """
    4 + 1/(2 + 1/(6 + 1/7)) is approximately 4/1, 9/2, 58/13 and 415/93

    >>> list(convergents_of_contfrac([4, 2, 6, 7]))
    [(4, 1), (9, 2), (58, 13), (415, 93)]
    """
    n0, n1 = cf[0], cf[0]*cf[1]+1
    d0, d1 = 1, cf[1]
    yield (n0, d0)
    yield (n1, d1)

    for i in range(2, len(cf)):
        n2, d2 = cf[i]*n1+n0, cf[i]*d1+d0
        yield (n2, d2)
        n0, n1 = n1, n2
        d0, d1 = d1, d2

def wieners_attack(e, n):
    cf = continued_fraction(e, n)
    convergents = convergents_of_contfrac(cf)

    for k, d in convergents:
        if k == 0:
            continue
        phi, rem = divmod(e*d-1, k)
        if rem != 0:
            continue
        s = n - phi + 1
        # check if x^2 - s*x + n = 0 has integer roots
        D = s*s - 4*n
        if D > 0 and gmpy.is_square(D):
            return d

e = 53529140412320991078599368841996837550696777383791310419022469164609033920207379533743700358356234039257393617026673654393163771456338610794570988203350657884943750738578896337344207227342285752214239863307356401404099138908990615473355323974066279917298836426381440724932512745433545023219719830791980975093
n = 60965526218693512543978708718661639861668920622865181011269408297116630276986445484298018705760779073576487243958033319221534749490538615220683642734702621798591077360257605162372614470122692218831538874529415599817899000924750479165823332025345266986320347701132497995102322602204978229917325387549699183397
c = 21432206656278282937320292486090681686194121441531047195732842962363381586535841811412180760129880383010878757931275630021189579218349735819053148657781384007902034646416528859758811247283036575211786278500060657615091673714992527822320115153978940837179729918645453373650761453915717250574916570194124691132

d = wieners_attack(e, n)
print(d)
m = pow(c, d, n)
print(codecs.decode(('%x'%m),'hex_codec'))

```

