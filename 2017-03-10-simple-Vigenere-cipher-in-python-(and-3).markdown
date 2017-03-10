Title: Simple Vigenere cipher in Python (and 3)
Date: 2017-03-10 13:13:17 +0200
Slug: 2017-03-10-simple-Vigenere-cipher-in-python-(and-3)
category: posts
tag: python, books, security

Last part of my series about Vigenere cipher. (3 post in a row? I am proud of myself :-P)

In my previous posts I already showed how to use Vigenere square to encryp/decypt text, so this time I'll follow the algebraic method described in the [Wikipedia](https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher#Algebraic_description):

{% img center https://dl.dropboxusercontent.com/u/14814182/blog/vigenere.jpg 'vigenere' %}

I'll use the same input, same key, and same alphabets as in previous execises:

```
*mykey* = "WHITE" 
*input_text* = "en un lugar dela mancha de cuyo nombre no quiero acordarme" 
*Position:*    		     	00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 
*Reference alphabet(M):*  	A  B  C  D  E  F  G  H  I  J  K  L  M  N  O  P  Q  R  S  T  U  V  W  X  Y  Z 
*Key alphabet(K):*		B  C  D  E  F  G  H  I  J  K  L  M  N  O  P  Q  R  S  T  U  V  W  X  Y  Z  A 
```

(We shifted the letters one position, as the author showed in the book, so K alphabet starts in "B" not in "A")

# Encryption:

Now we are going to use numbers instead of the square approach:

If you remember the first post, the foundation of this cipher is the tuple (letter,key):

```
INPUT: EN UN LUGAR DE LA MANCHA
KEY:   WH IT EWHIT EW HI TEWHIT
TUPLE: ('E', 'W'),('N', 'H'),('U', 'I'),('N', 'T'), ('L', 'E'),('U', 'W'),[...]
```

Let's get the positions of each element in the tuple fro M[] and K[]:

*1st tuple:* 'E' is in position 4 in our reference alphabet(M). 'W' is in position 21 in the Key alphabet(K) 
*2nd tuple:* 'N' is in position 13 in M[], 'H' is in position 6 in K[] 
*3rd tuple:* 'U' is in position 20 in M[], 'I' is in position 7 in K[] 

```
('4','21'),('13','6'),('20','7'),('13','18'),[...]
```

So putting this in the mathematical notation:

*C[i] = (M[i]+K[i]) mod len(M)*

```
C[0] = (4 + 21) % 26 = 25
C[1] = (13 + 6) % 26 = 19
C[2] = (20 + 7) % 26 = 1
[...]
```

So the letter "E" in postion 4 in M[] will be replaced by the letter in position 25 in K[], which is "A". The same way, the letter "N" in position 13 in M[] will be replaced by the letter in position 19 in K[], which is "U", and so on...

```
E -> K[25] ->A
N -> K[19] ->U
U -> K[1] ->C
N -> K[5] ->G
L -> K[14] ->P 
U -> K[15] ->Q
G -> K[12] ->N
[...]
```

# Decryption:

Decryption works pretty much the same way... The message is calculated this way:

*M[i] = (C[i]-K[i]) mod len(M)*

Let's check step by step.. This is our input data:

```
INPUT: AU CG PQNIK HA SI FEJJPT
KEY:   WH IT EWHIT EW HI TEWHIT
TUPLE: ('A', 'W'),('U', 'H'),('C', 'I'),('G', 'T'), ('P', 'E'),('Q', 'W'),('N', 'H'),('I', 'I'),('K', 'T'),('H', 'E'),[...]
```

We have to look for the positon of the each letter of each tuple in alphabet K[]:

*1st tuple:* Positions of letter "A" and "W" in K[], 25 and 21.
*2nd tuple:* Positions of letter "U" and "H" in K[], 19 and 6.
*3rd tuple:* C,I -> 1, 7

Now, coming back to the formula:

```
M[0] = (C[0]-K[0]) mod len(M) = (25 - 21) % 26 = 4 
M[1] = (C[1]-K[1]) mod len(M) = (19 - 6) % 26 = 13  
M[2] = (C[2]-K[2]) mod len(M) = (1 - 7) % 26 = 20 
M[3] = (C[3]-K[3]) mod len(M) = (5 - 18) % 26 = 13 
```

And now looking for those positions in our reference alphabet M[]:

```
A -> M[4] -> E 
U -> M[13] -> N 
C -> M[20] -> U 
G -> M[13] -> N 
P -> M[11] -> L 
Q -> M[20] -> U 
N -> M[6] -> G 
I -> M[0] -> A 
K -> M[17] -> R 
[...]
```

Let put this into python code:

```
#!/usr/bin/env python
import string

mykey="WHITE"
input_text="en un lugar de la mancha de cuyo nombre no quiero acordarme"
code_text="AU CG PQNIK HA SI FEJJPT HA JCRS JVUUVA UW JYELZH EYVZWENTM"

# Alphabet used as reference (M)
# ABCDEFGHIJKLMNOPQRSTUVWXYZ
source = string.ascii_uppercase

# Key alphabet (K) shifted 1 position to the left
# BCDEFGHIJKLMNOPQRSTUVWXYZA
shift = 1
matrix = [ source[(i + shift) % 26] for i in range(len(source)) ]

def coder(thistext):
	ciphertext = []
	control = 0

	for x,i in enumerate(input_text.upper()):
	    if i not in source: 
	    	#If the symbol is not in our reference alphabet, we simply print it
	        ciphertext.append(i)
	        continue
	    else:
	    	#Wrap around the mykey string 
	        control = 0 if control % len(mykey) == 0 else control 
	        
	        #Calculate the position C[i] = (M[i]+K[i]) mod len(M)
	        result = (source.find(i) + matrix.index(mykey[control])) % 26
	        
	        #Add the symbol in position "result" to be printed later
	        ciphertext.append(matrix[result])
	        control += 1
	
	return ciphertext

def decoder(thistext):
	control = 0
	plaintext = []

	for x,i in enumerate(code_text.upper()):
	    if i not in source: 
	        #If the symbol is not in our reference alphabet, we simply print it
	        plaintext.append(i)
	        continue
	    else:
	        #Wrap around the mykey string 
	        control = 0 if control % len(mykey) == 0 else control 
	   
	        #Calculate the position M[i] = (C[i]-K[i]) mod len(M)
	        result = (matrix.index(i) - matrix.index(mykey[control])) % 26

	        #Add the symbol in position "result" to be printed later
	        plaintext.append(source[result])
        	control += 1

	return plaintext

# Print results
print("Key: {0}".format(mykey))
print("\nDecode text:")
print("-> Input text: {0}".format(input_text))
print("-> Coded text: {0}".format(''.join(coder(input_text))))

# Print results
print("\nDecode text:")
print("-> Input text: {0}".format(code_text))
print("-> Decoded text: {0}".format(''.join(decoder(code_text)).lower()))
```

The code is pretty clean and simple to understand. There are two functions, and the key part is the calculation of *result* using the math formula shown above.

And see the result:

```
$ python Vigenere_cipher_mod.py
Key: WHITE

Decode text:
-> Input text: en un lugar de la mancha de cuyo nombre no quiero acordarme
-> Coded text: AU CG PQNIK HA SI FEJJPT HA JCRS JVUUVA UW JYELZH EYVZWENTM

Decode text:
-> Input text: AU CG PQNIK HA SI FEJJPT HA JCRS JVUUVA UW JYELZH EYVZWENTM
-> Decoded text: en un lugar de la mancha de cuyo nombre no quiero acordarme
```

There are tons of readings about how to break this code on the internet. I found these two very interesting:

* [Crypto Analysis to Crack Vigenere Ciphers](https://schoolcodebreaking.com/2015/06/18/crypto-analysis-to-crack-vigenere-ciphers/) 

* [Hacking the Vigenère Cipher](http://inventwithpython.com/hacking/chapter21.html)

Later

PS: I hate markdown
