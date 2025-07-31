---
title: RSA非对称加密
date: 2023-06-15 20:51:18
tags:
- 密码学
- 非对称加密
categories:
- 密码学
- 非对称加密

---

<center>
引言：RSA算法及RSA在代理重加密中的应用
</center>

<!-- more -->

# RSA算法

> RSA算法：由Ron Rivest、Adi Shamir和Leonard Adleman于1977年共同提出。这个算法基于数论中的数学问题，利用了大素数分解的困难性来提供安全性。

## RSA如何生成公钥与私钥？

1、首先我们需要两个大素数，比如p与q

```java
p = new BigInteger("851322...19074969");// 154个char
q = new BigInteger("836458...86991233");// 154个char
```

2、计算其乘积，设为n（n与加密解密有关）

```java
n = p * q
```

3、计算欧拉函数φ(n) = (p-1) * (q-1)，这里使用`PhiN`

```java
PhiN = p.subtract(BigInteger.valueOf(1));
PhiN = PhiN.multiply(q.subtract(BigInteger.valueOf(1)));
```

4、找一个**小于φ(n)且与φ(n)互质**的整数e，作为**公钥**的指数

```java
do {
    e = new BigInteger(2 * SIZE, new Random());
}
// 前面的条件表示 e>PhiN，后面的条件表示e与PhiN的最大公约数不为1，即表示不互质
// 当e>PhiN或e与PhiN不互质时就不断循环，直到两个条件都满足，即找到了一个小于φ(n)且与φ(n)互质的整数e
while ((e.compareTo(PhiN) != 1) || (e.gcd(PhiN).compareTo(BigInteger.valueOf(1)) != 0));
```

5、计算与e关于模φ(n)的**乘法逆元**d，作为**私钥**的指数。

```java
d = e.modInverse(PhiN);
```

即找到一个满足`(d*e)%PhiN==1`的d

这样我们就找到了**公钥的指数e**与**私钥的指数d**

e和d并不是公钥和私钥本身，公钥与私钥本身指的是**一个指数与一个模数**，在这里公钥的指数是e，模数是n；私钥的指数是d，模数也是n。

## RSA如何进行加密解密？

* 加密：使用公钥(e, n)，将明文数据m加密为密文c。加密操作为`c = m^e mod n`。
 * 解密：使用私钥(d, n)，将密文c解密为明文数据m。解密操作为`m = c^d mod n`。

## 代理重加密中使用RSA

在代理重加密的过程中，我们常用一个代理Proxy，代理使用重加密秘钥对已加密的数据进行第二次加密。

一种常见的重加密秘钥生成方式是：使用**发送者的私钥与接受者的公钥做乘积**

为什么使用乘积作为新的公钥去加密数据，接受者依然可以解密呢？

下面来证明一下其正确性：

首先我们需要知道基本的模运算法则和幂运算法则

![基本运算法则](http://img.yesmylord.cn//image-20230616103708954.png)

举个例子：

![接受者解密使用重加密秘钥加密的数据](http://img.yesmylord.cn//image-20230616103849990.png)



## 完整代码

```java
public class RSA {
    public BigInteger p, q;
    public BigInteger n;
    public BigInteger PhiN;
    public BigInteger e, d;

    public RSA() {
        Initialize();
    }

    //Generate e and d
    //e:- Public Key
    //d:- Private Key

    /**
     * 密钥生成：
     * 1 选择两个大素数p和q，并计算它们的乘积n，即n = p * q。
     * 2 计算欧拉函数φ(n)，即φ(n) = (p-1) * (q-1)。
     * 3 选择一个小于φ(n)且与φ(n)互质的整数e，作为公钥的指数。
     * 4 计算与e关于模φ(n)的乘法逆元d，作为私钥的指数。
     */
    public void Initialize() {

        int SIZE = 512;
        // p and q are 2 154 digit Prime Numbers which are used in the generation of RSA Keys
        p = new BigInteger("8513222065247162701695105220665738877312063308356937563625345485856710133446374665834898192825484459951443770023314504441479244278247980992441766519074969");
        q = new BigInteger("8364581280641288933593527550533091363060086128207408134848028170130641974184553465641962883238792572920670310338579332490687347012348067644317739328586993");
        n = p.multiply(q);
        PhiN = p.subtract(BigInteger.valueOf(1));
        PhiN = PhiN.multiply(q.subtract(BigInteger.valueOf(1)));
        do {
            e = new BigInteger(2 * SIZE, new Random());
        }
        // 前面的条件表示 e>PhiN，后面的条件表示e与PhiN的最大公约数不为1，即表示不互质
        // 当e>PhiN或e与PhiN不互质时就不断循环，直到两个条件都满足
        while ((e.compareTo(PhiN) != 1) || (e.gcd(PhiN).compareTo(BigInteger.valueOf(1)) != 0));
        d = e.modInverse(PhiN);
    }

    /**
     * 加密：使用接收者的公钥(e, n)，将明文数据m加密为密文c。加密操作为c = m^e mod n。
     * 解密：使用接收者的私钥(d, n)，将密文c解密为明文数据m。解密操作为m = c^d mod n。
     */
    public BigInteger encrypt(BigInteger plaintext) {
        return plaintext.modPow(e, n);
    }

    public BigInteger decrypt(BigInteger ciphertext) {
        return ciphertext.modPow(d, n);
    }
}
```

