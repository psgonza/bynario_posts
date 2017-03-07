Title: Simple Vigenere cipher in Python
Date: 2017-03-07 13:13:17 +0200
Slug: 2017-03-07-simple-Vigenere-cipher-in-python
category: posts
tag: python, books, security

I am currently reading ["The code book"](https://www.goodreads.com/book/show/17994.The_Code_Book) by Simon Singh, and he just described how the [Vigenere cipher](https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher) works... I am not coding any Python lately, so I have decided to implement it (real quick), not using any algorithm but manually, as someone would have done 300 years ago, preparing a Vigenere square, and then looking up the values in the table.

The Python code is pretty simple:

```
#!/usr/bin/env python
# Simple Vigenere cipher implementation in Python
import string

mykey="WHITE"
input_text="en un lugar de la mancha de cuyo nombre no quiero acordarme"

ciphertext = []
matrix = []
encryption_tuple= []
row = 0
control = 0

# Alphabet used as reference
source = string.ascii_uppercase

#Creating the Vigenere Square. A 26x26 matrix. 
# In the example provided by the book, instead of using the regular alphabet as reference, we shift the items, so 
# so the column used as reference doesn't start in A, but in B
for row in range(len(source)):
    matrix.append([ x for i,x in enumerate(source) if i > row ])   
    for i,x in enumerate(source):
        if i <= row: matrix[row].append(x)
      
# Creating the tuple based on the letter and key. ie:
# ('D', 'W'), ('I', 'H'), ('V', 'I'), ('E', 'T'), ('R', 'E'), ('T', 'W'), ('T', 'H'), ('R', 'I'), ...        
# In case special characters are not considered, this is cleaner:
#   import itertools
#   text=[ x for x in input_text.upper() if x in string.ascii_letters]
#   encryption_tuple = [(x,y) for x,y in zip(text, itertools.cycle(mykey))]
for x,y in enumerate(input_text.upper()):
    control = 0 if control % len(mykey) == 0 else control
    if y in string.punctuation or y in string.whitespace:
         encryption_tuple.append((y,y))
    else:
         encryption_tuple.append((y,mykey[control]))
         control += 1

# Each element y in the tuple is the key in the alphabet matrix
for x,y in encryption_tuple:
    if source.find(x) == -1: 
        ciphertext.append(x)
    else:
        ref_row = matrix[0].index(y)
        ciphertext.append(matrix[ref_row][source.index(x)])

# Print results
print("-> Square:")        
for i in matrix:
    print(' '.join(i))
print("-> Key: {0}".format(mykey))
print("-> Input text: {0}".format(input_text))
print("-> Output text: {0}".format(''.join(ciphertext)))
```

I stored the code in [GITHUB](https://github.com/psgonza/bynario/blob/master/simple_vinegere_cipher.py).

Not very likely to replace [opengpg](https://gnupg.org/), but it works:

```
$ python vigenere_cipher.py
-> Square:
B C D E F G H I J K L M N O P Q R S T U V W X Y Z A
C D E F G H I J K L M N O P Q R S T U V W X Y Z A B
D E F G H I J K L M N O P Q R S T U V W X Y Z A B C
E F G H I J K L M N O P Q R S T U V W X Y Z A B C D
F G H I J K L M N O P Q R S T U V W X Y Z A B C D E
G H I J K L M N O P Q R S T U V W X Y Z A B C D E F
H I J K L M N O P Q R S T U V W X Y Z A B C D E F G
I J K L M N O P Q R S T U V W X Y Z A B C D E F G H
J K L M N O P Q R S T U V W X Y Z A B C D E F G H I
K L M N O P Q R S T U V W X Y Z A B C D E F G H I J
L M N O P Q R S T U V W X Y Z A B C D E F G H I J K
M N O P Q R S T U V W X Y Z A B C D E F G H I J K L
N O P Q R S T U V W X Y Z A B C D E F G H I J K L M
O P Q R S T U V W X Y Z A B C D E F G H I J K L M N
P Q R S T U V W X Y Z A B C D E F G H I J K L M N O
Q R S T U V W X Y Z A B C D E F G H I J K L M N O P
R S T U V W X Y Z A B C D E F G H I J K L M N O P Q
S T U V W X Y Z A B C D E F G H I J K L M N O P Q R
T U V W X Y Z A B C D E F G H I J K L M N O P Q R S
U V W X Y Z A B C D E F G H I J K L M N O P Q R S T
V W X Y Z A B C D E F G H I J K L M N O P Q R S T U
W X Y Z A B C D E F G H I J K L M N O P Q R S T U V
X Y Z A B C D E F G H I J K L M N O P Q R S T U V W
Y Z A B C D E F G H I J K L M N O P Q R S T U V W X
Z A B C D E F G H I J K L M N O P Q R S T U V W X Y
A B C D E F G H I J K L M N O P Q R S T U V W X Y Z
-> Key: WHITE
-> Input text: en un lugar de la mancha de cuyo nombre no quiero acordarme
-> Output text: AU CG PQNIK HA SI FEJJPT HA JCRS JVUUVA UW JYELZH EYVZWENTM
```

I'll try to find time to code the decryption part and also to try different ways of implementing it.

By the way, I totally recommend the book!

Later
