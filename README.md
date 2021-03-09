# Security for The Internet Of Things
*Building a small security library, compatible with the resources of the Internet of Things, while integrating the ins and outs of security in this field*

authors:
  * [Bogdan Gorelkin](https://b.gorelkin.me)  <bogdan@gorelkin.me>
  * [Sheikh Shah Mohammad Motiur Rahman](https://motiur.info) <motiur@ieee.org>

supervisor:
  * [Christophe Guyeux](https://www.femto-st.fr/fr/personnel-femto/cguyeux) <guyeux@gmail.com>

project relaized on [pyBoard](https://store.micropython.org/product/PYBv1.1H)

---
<a name="menu"></a>
###  The different stages of the constitution of the library:

1. [A first symmetrical encryption algorithm](#1)
2. [Constitution of a random generator](#2)
3. [How to efficiently calculate a power](#3) 
4. [Generation of prime numbers](#4)
5. [Diffie–Hellman key exchange](#5)
6. [Asymmetric encryption algorithms:](#6)
   1. [RSA](#6_1) </br>
   2. [El Gamal](#6_2) 

###  1. A Symmetrical Encryption Algorithm for the IoT<a name="1"></a>[↩](#menu)
Suppose you want to encrypt a message you have in the form of a sequence of bits:

```0010010100011110011111110```

and that one has moreover a pseudo-random number generator at disposal. We will generate as many random bits as there are in the message to be encrypted:

```1011000101110010001110100```

We can hide our message in this noise by the technique known as the "one-time pad", where the cryptogram is the bit-by-bit exclusive OR (or, in an equivalent way, the modulo 2 addition) between the original message and this random sequence:

```       
       0010010100011110011111110
   XOR 1011000101110010001110100
     = 1001010001101100010001010
```

Our cryptogram is therefore:

```1001010001101100010001010```


Alice sends this message to Bob who, it is assumed, is capable of generating exactly the same noise:

```1011000101110010001110100```


It performs the same operation: bit by bit exclusive OR between the cryptogram and this noise:

``` 
        1001010001101100010001010
    XOR 1011000101110010001110100
      = 0010010100011110011111110
```

We fall back, as we can see, on the original message. In fact, as this is the modulo 2 addition, and that we added twice our sequence of noise bits, we added either 0 or 2, the whole modulo 2: so we obviously go back to the original sequence.

### 2. Constitution of Random Generators<a name="2"></a>[↩](#menu)
Random and pseudo-random number generators are one of the foundations of computer security.

The first ones are based on hardware:</br>
* They use a physical source of entropy, using the sensors of the device: kth decimal place of the processor temperature, background noise picked up by the microphone, pressure, values picked up by the accelerometer, etc.
* These can be mixed with the memory residue, possibly with the user's action
* Based on a physical noise, it is a "true" random variable, but not provable: for example, we cannot prove that the random variable corresponding to this physical generation follows a normal law.
* This random is not reproducible.
* In cryptography, such generators are necessary to generate keys for cryptosystems.
* With pyboard, we have access to such a generator: pyb.rng().

The second are based on algorithms:</br>
* Most of the time, the starting point is a modulo N recurrent sequence.
* The sequence is chosen so that the numbers produced during the iterations look a lot like random numbers.
* By "resemble", we mean for example: no statistical test should be able to distinguish between a truly random sequence and a sequence produced by this algorithm.
* The first term of the sequence is the seed of the pseudo-random generator. Providing the same seed twice in a row leads to generating the same pseudo-random numbers twice in a row.
* This reproducibility is interesting in cryptography: it makes it possible to easily perform symmetrical encryption by masking in noise, as we have seen previously.

###### LINEAR CONGRUENTIAL GENERATORS
We present in the following some of the generators actually used, starting with the linear congruential generators (LCG, 1948):</br>
* Their very low complexity makes them usable in the Internet of Things, even at very low resources;
* They are however unsafe, so other generators are later introduced.
These LCGs are defined as follows:
``` 
Xn+1=(a⋅Xn+c)%m
``` 
where `a`, `c` and `m` are the generator parameters.

These generators are obviously periodic, and some parameters produce better sequences than others, in the sense that their periods are maximal. To obtain such parameters, it is necessary and sufficient that:</br>
* `c` and `m` are coprime integers (their gcd is 1).
* For each prime number `p` dividing `m, (a−1)` is a multiple of `p`.
* `m` multiple of `4 ⇒(a−1)` multiple of 4.</br>
(valid for c≠0).

###### THE XORSHIFT
This generator has been proposed recently by Marsaglia, it has basically three integer parameters `a,b,c`. Its operation can be summarized as follows:</br>
* We start from a 32-bit vector, which serves as a seed.
* To calculate the new term from the current term `x`:
    * We replace x by the result of the or exclusive bit-by-bit between x and its version shifted `a` bits to the right (circular shift, the bits going out to the right go in to the left).
    * Replace x with the result of the or exclusive bit-to-bit between x and its version shifted `b` bits to the left (circular shift).
    * We replace x by the result of the or exclusive bit-to-bit between x and its version shifted `c` bits to the right (circular shift).
* The value obtained after these three operations is the new term of the sequence, to be output.

A piece of code in C detailing the XORshift with (12, 25, 27) as parameters would therefore be:
``` 
  x ^= x >> 12; // a
  x ^= x << 25; // b
  x ^= x >> 27; // c
  ``` 
### 3. How to efficiently calculate a power <a name="3"></a>[↩](#menu)
Fast exponentiation is a classical technique, used in practice, to obtain the power of a given number.

This fast exponentiation iterates, for the calculation of **X<sup>e</sup>**, the following operations:
* The square elevation,
* Multiplication by **X**,
in an order depending on the base **2** entry of **e**.

It is well understood in its recursive version :
* If e is even, we calculate (recursively) **X<sup>e/2</sup>**, which we multiply by itself,
* Otherwise, we calculate (recursively) **X<sup>e-1</sup>**, which we multiply by **X**.

###### THE ALGORITHM
As the above technique is recursive, its implementation in a sensor may be problematic. But as the operations to be done are determined by the parity of the number following a division by **2**, or has a division by **2** after a deletion of **1**, we deduce that the binary writing of the number contains exactly the operations to be done.

More precisely...
* We write **e** in base **2**,
* We remove the first **1** of this writing,
* One replaces then, in this writing,
    * the **0** by **S**,
    * the **1** by **SX**,
    we thus obtain a word made up of the letters **S** and **X**.
* We start from the number **X**
    * if the first letter is **S**, we raise to the square
    * otherwise, multiply by **X**.
* Start again with the following letters, until the end of the word.

### Generation of prime numbers <a name="4"></a>[↩](#menu)
###### ERATOSTHENES' SIEVE
To obtain a list of prime numbers below **N**, one can proceed as follows:
* Create the list of integers from **2** to **N**,
* Delete, in this list, the multiples of **2**, then the multiples of the next remaining number in the list **(3)**, then the multiples of the next remaining number in the list **(5)**...
* Stop at **N/2**.

######## A FERMAT THEOREM
*Small theorem of Fermat*</br>
If **n** is prime, then **n** divides **a<sup>n</sup>−a,a>1**.</br>
In other words, **a<sup>n</sup>≡a[n]**.
######## APPLICATION TO PRIMALITY TESTS
To obtain a large prime number, the idea is to draw a large number of digits, concatenate them as a number, and then test if this number is prime.

The problem with Fermat's theorem is that the implication is in the wrong direction: one would want something of the form "if such and such a property, then the number is prime", as for example in Wilson's theorem. However :
* It is difficult to be more efficient than Fermat.
* If the reciprocal is false in general, it is concretely very often true, and can be used as the basis of a probabilistic method of primality testing: something that, when it says that the number is not prime, is never wrong (a proven meaning of Fermat's theorem), and when it says that the number is prime, has a very small risk of being wrong.
### Diffie–Hellman key exchange <a name="5"></a>[↩](#menu)
Read more about Diffie–Hellman key exchange [here](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange)
### Asymmetric encryption algorithms: <a name="5"></a>[↩](#menu)
###### RSA <a name="6_1"></a>[↩](#menu)
Read more about RSA encryption [here](https://en.wikipedia.org/wiki/RSA_(cryptosystem))
###### El Gamal <a name="6_2"></a>[↩](#menu)
Read more about El Gamal encryption [here](https://en.wikipedia.org/wiki/ElGamal_encryption)
