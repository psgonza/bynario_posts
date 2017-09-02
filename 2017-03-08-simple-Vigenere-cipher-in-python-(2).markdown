Title: Simple Vigenere cipher in Python (2)
Date: 2017-03-08 13:13:17 +0200
Slug: 2017-03-08-simple-Vigenere-cipher-in-python-(2)
category: posts
tags: python, books, security

See:
[Part 3/3](https://bynario.com/2017-03-10-simple-Vigenere-cipher-in-python-(and-3).html)

Just a small update to my previous post about the [Vigenere cipher](https://bynario.com/2017-03-07-simple-Vigenere-cipher-in-python.html)

Following the same approach as in the cipher, I modified a few lines in the script to create the "decoder"... I won't paste the code here, because the script is 95% the same, but I have stored it in [GITHUB](https://github.com/psgonza/bynario/blob/master/simple_vinegere_cipher_decoder.py).

Basically, some variables change:

* coder: ```input_text="en un lugar de la mancha de cuyo nombre no quiero acordarme" ```
* decoder: ```input_text="AU CG PQNIK HA SI FEJJPT HA JCRS JVUUVA UW JYELZH EYVZWENTM" ```

* coder: ```ciphertext = [] ```
* decoder: ```cleartext = [] ```

* coder: ```print("-> Output text: {0}".format(''.join(ciphertext))) ```
* decoder: ```print("-> Output text: {0}".format(''.join(cleartext))) ```

And the logic for the look up changes as well:

In the coder, we replace the letter in the input_text by the one in the matrix[n] :

```
for x,y in encryption_tuple:
    if source.find(x) == -1: 
        ciphertext.append(x)
    else:
        ref_row = matrix[0].index(y)
        ciphertext.append(matrix[ref_row][source.index(x)])
```

In the decoder, we look for the position (lets call it "y") of the letter in matrix[n] and replace it by the letter in the position "y" in the alphabet:

```
for x,y in encryption_tuple:
    if source.find(x) == -1: 
        cleartext.append(x)
    else:
        ref_row = matrix[0].index(y)
        cleartext.append(source[matrix[ref_row].index(x)])
```        
   
The result is the expected:   

```
-> Key: WHITE
-> Input text: AU CG PQNIK HA SI FEJJPT HA JCRS JVUUVA UW JYELZH EYVZWENTM
-> Output text: EN UN LUGAR DE LA MANCHA DE CUYO NOMBRE NO QUIERO ACORDARME
```

In the next (and last post) about Vigenere, I'll write another simple coder/decoder script but based in the [mathematical  concept](https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher#Algebraic_description)

Laters!
