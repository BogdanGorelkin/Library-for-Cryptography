# Security for The Internet Of Things
*Building a small security library, compatible with the resources of the Internet of Things, while integrating the ins and outs of security in this field*

authors:
  * [Bogdan Gorelkin](https://b.gorelkin.me)  <bogdan@gorelkin.me>
  * [Sheikh Shah Mohammad Motiur Rahman](https://motiur.info) <motiur@ieee.org>

supervisor:
  * [Christophe Guyeux](https://www.femto-st.fr/fr/personnel-femto/cguyeux) <guyeux@gmail.com>

project relaized on [pyBoard](https://store.micropython.org/product/PYBv1.1H)

---

###  The different stages of the constitution of the library:

1. A first symmetrical encryption algorithm
2. Constitution of a random generator
3. How to efficiently calculate a power 
4. Generation of prime numbers
5. Asymmetric encryption algorithms:
> 5.1. RSA
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

### 2. Constitution of Random Generators
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
