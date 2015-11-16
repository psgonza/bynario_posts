title: Does knowing all the idioms/built-in functions make you a better coder?
date: 2015-11-11 11:11:11
slug: 2015-11-11-does-knowing-all-the-idioms-make-you-a-better-coder
tag: python, coding, opinion
category: posts
status: draft

(Disclaimer: I do not code for a living...)

I like coding in Python, it is my go-to language for almost everything nowadays. Easy to learn, pretty popular and fast enough, so no complains, but sometimes python idioms and built-in function/methods drive me nuts… 

I understand you cannot be a good python developer if you don’t know your language, but unless you write python code 24/7, I think we shouldn’t focus on writing 100% pythonic code…. It is not worth it.

Let me put a very basic example. Not sure if you know about [checkio](http://www.checkio.org/), a game/community/platform for learning (or improving) python (100% recommended by the way), where you have hundreds of “missions” (tasks to be solved by using Python), and you can share your solutions with the rest of the community…  This is the most interesting part by far, because you can compare your code, and (in my case) realize how bad it is 

For example, one of the missions asks you to write the code for transposing any given matrix, which could be easily achieved  in 10 lines if you understand the concept:

Python:
```
matrix =[[1,2,3], [4,5,6], [7,8,9]]
transpose = []
for j in range(len(matrix[0])):
    row=[]
    for i in matrix:
        row.append(i[j])
    transpose.append(row)
```

Yes, the code is simple, not optimal, and can be improved a lot, but it  can be translated to C in 1 minute:

C:
```
int j = 3;
int k = 3;
int c, d;
int matrix[3][3]={{1,2,3},{4,5,6},{7,8,9}};
int transpose[3][3];

for (c = 0; c < j; c++)
  for( d = 0 ; d < k ; d++ )
     transpose[d][c] = matrix[c][d];
```

On the other hand, if I were a python developer, I could come up with something like this:

Python using built-in functions:
```
matrix =[[1,2,3], [4,5,6], [7,8,9]]
transpose = list(map(list, zip(*matrix)))
```

[(solution by PositronicLlama)](http://www.checkio.org/mission/matrix-transpose/publications/PositronicLlama/python-3/functional/)

Ok, I love one-liners as much as the next guy and  I agree that is a great Pythonic answer… But it makes me wonder:

* __Is that code easy to  read and understand?__  I don’t think anyone with basic/average python skills can understand it.
* __When performance is not an issue, why not writing a small piece of code instead of relying in a function you don't even know how works?__  I am not a python developer (truth being told, I am not even a developer), so for me it is more interesting to understand how to solve a problem than solving it…  Because I can use the same approach in any other language if needed.
* __Are we better developers if we use list/map/zip in the same line?__ Probably the short answer is yes. I think there is no such thing as “generalist developer”, most of people know a lot about just one language, and if you are really (really) good, you could know 2 or maybe 3 , but that is not that common… So yes, if you want to be a “pr0” python programmer, you would need to know all these python idioms, built-in functions and modules that makes Python so great. 

For the “n00b”/average  programmers as myself, yeah, they are cool, save a few lines and are damn fast so I use some of them, but you know,  it is not that important.
