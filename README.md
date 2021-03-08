# Security for The Internet Of Things
*Building a small security library, compatible with the resources of the Internet of Things, while integrating the ins and outs of security in this field*

authors:
  * [Bogdan Gorelkin](https://b.gorelkin.me)  <bogdan@gorelkin.me>
  * [Sheikh Shah Mohammad Motiur Rahman](https://motiur.info) <motiur@ieee.org>

supervisor:
  * [Christophe Guyeux](https://www.femto-st.fr/fr/personnel-femto/cguyeux) <guyeux@gmail.com>

project relaized on [pyBoard](https://store.micropython.org/product/PYBv1.1H)

---

##  The different stages of the constitution of the library:

1. A first symmetrical encryption algorithm
2. Constitution of a random generator
3. How to efficiently calculate a power
4. Generation of prime numbers
5. Asymmetric encryption algorithms:
> 5.1. RSA</br>
> 5.2. El Gamal

###  1. A Symmetrical Encryption Algorithm for the IoT
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


