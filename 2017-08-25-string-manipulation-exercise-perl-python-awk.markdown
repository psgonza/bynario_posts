Title: String manipulation exercise: Perl, Python, Awk
Date: 2017-08-25 08:25:00 +0200
Slug: 2017-08-25-string-manipulation-exercise-perl-python-awk
category: posts
tag: programming,python,perl,awk


Here comes a small comparation of the performance of Perl, Awk and Python while parsing and splitting lines in a BIG ldif file with thousands or millions of subscriber profiles like [this one](https://raw.githubusercontent.com/psgonza/bynario/master/2017-08-25-string-manipulation_example.ldif) (I created this ldif file as an example, just for the sake of clarity)

As in the example, one of the attributes in my ldif files was a huge base64 string (more than 10k bytes long) in a single line, which is not supported by slapadd/slapd (I have checked LDIF rfc and I don't see any mention to the 4096 bytes limitation, so it could be our own implementation, not sure about this), so the idea was to spit this line into 76 characters lines (as per recommendation)... Something like this:

Original line (short version):

```
...
Service: SSBhbSBoYXBweSB0byBqb2luIHdpdGggeW91IHRvZGF5IGluIHdoYXQgd2lsbCBnbyBkb3duIGluIGhpc3RvcnkgYXMgdGhlIGdyZWF0ZXN0IGRlbW9uc3RyYXRpIGV2ZXJ5IHN0YXRlIGZa
...

```

Replaced by:

```
...
Service: SSBhbSBoYXBweSB0byBqb2luIHdpdGggeW91IHRvZGF5IGluIHdoYXQgd2lsbCBnbyBkb3
 duIGluIGhpc3RvcnkgYXMgdGhlIGdyZWF0ZXN0IGRlbW9uc3RyYXRpIGV2ZXJ5IHN0YXRlIGZa
...
```

(Notice the blank at the beginning of the second line)

My first approach was to use Python standard module *textwrap*:

```
import sys
import textwrap

try:
    ldiffile = sys.argv[1]
except:
    print("Input file missing... Exit")
    sys.exit(0)


with open(ldiffile, "rU") as f:
    for line in f:
        if line.startswith("Service:"):
            print(textwrap.fill(line, width=76,subsequent_indent=' '))
        else:
            print(line.strip())
```

It was really simple and produced the expected result, but it was ridiculously slow. **RIDICULOUSLY**. Really, difficult to believe...

So I decided to do something similar, but this time I dealt with the line split myself:

```
import sys

try:
    ldiffile = sys.argv[1]
except:
    print("Input file missing... Exit")
    sys.exit(0)

def fixlen(s, n):
    #First line, no blank at the beginning of the line
    print (s[:n])
    s = s[n:]
    #now, starting with blank
    while s:
        print (" " + s[:n])
        s = s[n:]

with open(ldiffile, "r") as f:
    for line in f:
        if line.startswith("Service:"):
            fixlen(line,76)
        else:
            print(line.strip())
```

The results I got were way better, but still, it took more than I expected... Time to give it a try with other tools: AWK and Perl

Basically I "translated" the python script to awk and perl, almost to the letter...

Perl version:

```
#!/usr/bin/perl
my $noident = 0;
while ($line = <>) {
  chomp($line);
  if ( $line =~ /^Service:.*/)
  {
    for (unpack("(A76)*",$line)) {
        if ($noindent == 0) {
            print "$_\n";
            $noindent = 1; }
        else { print " $_\n"; }
    }
  }
  else
  {
    $noindent = 0;
    print "$line\n";
  }
}
```

Awk version:

```
awk '
  BEGIN {
    indent = 0;
    i=0;
  }
  {
  if ( $0 ~ /^Service:.*/) {
    while(i<=length($0)){
        if ( $indent == 0 ) { printf "%s\n", substr($0,i,76);i+=76;indent=1; }
        else { printf " %s\n", substr($0,i,76);i+=76;}
   }
   }
   else {
        print $0;
   }
  }' $1

```

I "faked" several input files with thousands of lines (discarding the output), executed all the scripts in Cygwin64, and then checked the time it took.. Something like this:

```
time python3 textwrap.py XXXX.ldif &> /dev/null
time python3 pythonv1.py XXXX.ldif &> /dev/null
time python3 pythonv2.py XXXX.ldif &> /dev/null
time perl split.pl  XXXX.ldif &> /dev/null
time ./split.awk XXXX.ldif &> /dev/null
```

All of them generated the exact same output (except the textwrap version that handles the first line in a different way), but the execution time differs:

{% img center https://raw.githubusercontent.com/psgonza/bynario/master/results_table.JPG 'results_table' %}

It is easier to see in a chart:

{% img center https://raw.githubusercontent.com/psgonza/bynario/master/results_chart.JPG 'results_chart' %}

My takeaways after this small exercise:

- Awk rocks. 
- Perl's black magic is almost as fast as awk. 
- Python is not really fast at file processing. Yes, there are ways to improve this, by splitting in chunks, parallel processing and whatnot... But it takes more than 15 lines. 
- Stay away of textwrap module for heavy usage. 

It was fun... Bye!

PS: As you can see in the results, there is a "Python v2" column which produced better results... It is a modification of the original script where I used a comprehension list approach:

```
import sys

try:
    ldiffile = sys.argv[1]
except:
    print("Input file missing... Exit")
    sys.exit(0)

def fixlen(s, n):
    first = True
    tmp = (s[0+i:n+i] for i in range(0, len(s), n))
    for x in tmp:
        #First line, no blank at the beginning of the line
        if first:
            print(x)
            first=False
        else:
            #now, starting with blank
            print(" " + x)

with open(ldiffile, "r") as f:
    for line in f:
        if line.startswith("Service:"):
            fixlen(line,76)
        else:
            print(line.strip())
```
