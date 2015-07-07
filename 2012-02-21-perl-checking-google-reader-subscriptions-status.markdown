title: PERL: Checking Google Reader subscriptions status
slug: 2012-02-21-perl-checking-google-reader-subscriptions-status
date: 2012-02-21 08:53:00
category: posts
tag:  scripts,PERL

RSS subscriptions in my Google Reader account have reached an indecent number. I am subscribed to 398 sites (blogs, forums or whatever).

I will need to unsubscribe for a few of them I am no longer interested in, but I had the feeling that a lot of the feeds in my list weren't active at all.

Given the number of subscriptions, I wasn't too keen on checking each link manually, so I came up with this perl script.

I haven't use any module, so it could be executed with the default Perl environment that comes "out-of-the-box". The only requirements are wget, curl , the your subscriptions exported into a OPLM file and  internet connectivity :)

I have tested it under Cygwin 1.7 and Arch Linux, but any other GN/Linux distro should work fine as well.

The execution is pretty straightforward:

```
$ perl check_google_subscriptions.pl google-reader-subscriptions.xml
```

The script will look for duplicated items in your file, and will go through all your subscriptions, looking for the http response.

This the source code:

```
#!/usr/bin/perl
#######################################################
#Author: psgonza
#Date: 19-Feb-2012
#Objetive: Check Google Reader feed links status
#Version:1
#######################################################
use FileHandle;
use strict;
use warnings;
binmode STDOUT, ":utf8";
$| = 1;
die "[ERR] Usage: $0 <file.xml>\n" unless @ARGV;
my $wget = system ("which wget > /dev/null 2>&1");
if ($wget ne 0) {print "[ERR] wget not found\n"; exit 0;}
my $curl = system ("which curl > /dev/null 2>&1");
if ($curl ne 0) {print "[ERR] curl not found\n"; exit 0;}
my $googlexml = $ARGV[0];
my @lines;
my @new_lines;


sub check_link{
        my $urlList = shift;
        my @wrongUrl = ();
        my $counter = 1;
        print "$counter";
        foreach my $url(@$urlList){
        my $result = system ("wget --spider \"$url\" --server-response --timeout=5 --tries=3 > /dev/null 2>&1");
        if ($result != 0) { push (@wrongUrl, $url); }
        if ($counter % 10) {print ".";} else {print "$counter";}
        $counter++;    
        }
        printf " DONE!\n\n";
        return \@wrongUrl;
}
print "==============================\n";
#Open xml file
open FILE, $googlexml or die $!;
#Create array with feeds urls
while (<FILE>)
{
        if ($_ =~ /xmlUrl=\"(.*)\" /) { push (@lines, $1);}
}
#Close xml file
close FILE;
my $total_items = scalar(@lines);
print "$total_items links found in $googlexml\n";
#Create hash with uniqe items
my %tmp = map { $_, 0 } @lines;
my @links = sort(keys %tmp);

#Check for duplicated items
print "\n===========================================\n";
print "Duplicated links:\n";
print "===========================================\n";
my %cnt;
$cnt{$_}++ for @lines;
print "$_\n" for grep $cnt{$_} > 1, keys %cnt;



my $items = scalar(@links);
print "\n==============================================================\n";
print "Checking server response for $items items. It can take a while...\n";
print "===============================================================\n\n";
my $ref_list = check_link(\@links);
my $deadlink = scalar(@$ref_list);
if ( $deadlink ne 0 )
{
        print "$deadlink problematic links found\n\n";
}
else
{
        print "\nAll links working\n";
        print "\n===========================================\n";
        print "DONE\n";
        print "===========================================\n";
        undef($ref_list);
        exit 0;
}
print "\n===========================================\n";
print "Those problematic links could fail due to:\n\n";
print "- RSS not available in the server any more\n";
print "- Spiders are blocked in Robots.txt so the query fails\n\n";
print "Downloading the headers using curl.";
foreach my $val(@$ref_list){
        print ".";
        chomp $val;
        if (my $fineresult = system ("curl --connect-timeout 5 --max-time 5 --dump-header headers.txt \"$val\" > /dev/null 2>&1 && (cat headers.txt | grep \"HTTP/1.1 200 OK\" > /dev/null)") != 0) 
                { push (@new_lines, $val); }
}
print ". DONE! \n\n";
$deadlink = scalar(@new_lines);
if ( $deadlink ne 0 )
        {
        print "\n===========================================\n";
        print "$deadlink links found not available\n\n";
        }
        else {
        print "All links working\n";
        }
foreach my $val(@new_lines){
        print "--> $val <-- NOT AVAILABLE\n";
}

undef($ref_list);
print "\n===========================================\n";
print "DONE\n";
print "===========================================\n";
```

And this how the output will look like:

```
$ perl check_google_subscriptions.pl example_google_export
==============================
60 links found in example_google_export
===========================================
Duplicated links:
===========================================
http://www.xtorquemadax.com/feeds/posts/default?alt=rss
==============================================================
Checking server response for 59 items. It can take a while...
===============================================================
1.........10.........20.........30.........40.........50......... DONE!
7 problematic links found

===========================================
Those problematic links could fail due to:
- RSS not available in the server any more
- Spiders are blocked in Robots.txt so the query fails
Downloading the headers using curl......... DONE!

===========================================
7 links found not available
--> http://blog.fredjean.net/articles.atom
--> http://blogs.elcorreo.com/el-navegante/posts.rss
--> http://feeds.feedburner.com/github
--> http://veejoe.net/blog/feed/
--> http://www.irfree.com/feed/rss/
--> http://www.misionurbana.com/site/index.php?q=rss.xml
--> http://www.rsdownload.net/option,com_rss/feed,RSS2.0/no_html,1.html
===========================================
DONE
===========================================
```


It's not big deal, but does the job!

Enjoy :)
