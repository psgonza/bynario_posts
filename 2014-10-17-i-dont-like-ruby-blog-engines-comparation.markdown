title: Blog engines comparation
date: 2014-10-17 16:11:51 +0200
slug: 2014-10-17-i-dont-like-ruby-blog-engines-comparation
tags: coding, bynario, blogs

>> I don't like Ruby, at all... Maybe because I never really "used it" so didn't have the opportunity to know its guts, or maybe because it just sucks... If you ask me, it is probably the latter :P 

As you might know, Octopress (which is a great blog engine, kudos to [Brandon Mathis](https://twitter.com/imathis)) is written in Ruby, so every now and then I have to deal with rvm, and all the ruby paraphernaliam when an upgrade breaks it in my VM. 

Working in an environment you are not familiar with is not easy and every time it breaks, I have to spend some time trouble-shooting it, so I have decided to take a look at other available blog engines, compare them with Octopress, and migrate to a more "friendly" environment (hopefully)

So I compared these three alternatives: [Octopress](http://octopress.org/), [Tinkerer](http://tinkerer.me/) and [Hugo](http://gohugo.io/)

Below you can see some facts and personal opinions about the three engines:

* Octopress: 
    + Written in Ruby
    + Based on Jekyll
    + Ready to use out of the box. (version 3.0 in progress)
    + Installation pretty well documented, but requieres setting up ruby environment
    + Not that really fast... at all
    + Very mature (it's been around for 3 o 4 years)
    + Add-on plugins
    + 8380 stars in [GitHub](https://github.com/imathis/octopress)

{% img center https://dl.dropboxusercontent.com/u/14814182/blog/octopress_Capture.JPG octopress %}
    
* Tinkerer:
    + Written in Python
    + Based on Sphinx
    + No markdown
    + It needs a few dependences (like python-dev libxml2-dev libxslt-dev lib32z1-dev libz-dev)
    + Fast
    + 156 stars in [GitHub](https://github.com/vladris/tinkerer)

{% img center https://dl.dropboxusercontent.com/u/14814182/blog/tinker_Capture.JPG tinkerer %}    

* Hugo:
    + Written Golang
    + Using its own generation engine
    + Highly customizable. Barely usable right after installation
    + Run in multiple OS (including Windows)
    + Really easy to install. Binary available for several OS
    + Oh boy this is FAST.
    + 2239 stars in [GitHub](https://github.com/spf13/hugo)

{% img center https://dl.dropboxusercontent.com/u/14814182/blog/hugo_Capture.JPG hugo %}

Besides installing and playing around with them, I decided to run a small benchmark, just to compare the performance of the different engines. Basically this is what I did:

1 - Generate 500 random posts:

``` 
$ time for i in `seq 1 500`; do rake new_post["post $RANDOM $RANDOM"];done
Creating new post: source/_posts/2014-10-15-post-25028-3011.markdown
mkdir -p source/_posts
Creating new post: source/_posts/2014-10-15-post-16477-16246.markdown
mkdir -p source/_posts
[...]
Creating new post: source/_posts/2014-10-15-post-1586-30872.markdown
mkdir -p source/_posts
Creating new post: source/_posts/2014-10-15-post-680-27529.markdown

real    7m35.910s
user    6m53.838s
sys     0m38.886s

```

``` 
$ time for i in `seq 1 500`; do tinker --post "post $RANDOM";done
New post created as '/root/bynario/2014/10/15/post_15124.rst'
New post created as '/root/bynario/2014/10/15/post_19047.rst'
[...]
New post created as '/root/bynario/2014/10/15/post_14905.rst'
New post created as '/root/bynario/2014/10/15/post_10370.rst'
New post created as '/root/bynario/2014/10/15/post_4049.rst'

real    2m18.046s
user    1m55.815s
sys     0m18.737s

```

```
$ time for i in `seq 1 500`; do hugo new post/post_${RANDOM}_${RANDOM}.md;done
/root/hugoblog/content/post/post_24649_29646.md created
/root/hugoblog/content/post/post_24308_8060.md created
/root/hugoblog/content/post/post_8311_6440.md created
[...]
/root/hugoblog/content/post/post_12973_1855.md created
/root/hugoblog/content/post/post_23049_13857.md created
/root/hugoblog/content/post/post_26136_21085.md created

real    0m15.123s
user    0m7.424s
sys     0m5.688s

```

2 - Add some text (no markdown, just 50 lines of plain text) to each post in order not to generate (almost) empty html pages

3 - Build the whole site
``` 
$ time rake generate
## Generating Site with Jekyll
identical source/stylesheets/screen.css
Configuration file: /root/octopress/_config.yml
            Source: source
       Destination: public
      Generating... done.
 Auto-regeneration: disabled. Use --watch to enable.

real    2m45.143s
user    2m41.834s
sys     0m3.172s
```

``` 
$ time tinker --build
Making output directory...
Running Sphinx v1.2.3
loading pickled environment... not yet created
building [html]: targets for 502 source files that are out of date
updating environment: 502 added, 0 changed, 0 removed
reading sources... [100%] master
looking for now-outdated files... none found
pickling environment... done
checking consistency... done
preparing documents... done
writing output... [100%] master
writing additional files... search
copying static files... WARNING: favicon file 'tinkerer.ico' does not exist
done
copying extra files... done
dumping search index... done
dumping object inventory... done
build succeeded, 1 warning.

real    0m59.897s
user    0m57.792s
sys     0m1.328s
```

``` 
$ time hugo  --theme=liquorice --buildDrafts
498 of 498 drafts rendered
0 future content
498 pages created
0 categories created
0 tags created
in 2098 ms

real    0m2.136s
user    0m1.860s
sys     0m0.252s
```

Let's put the results in a table... I think figures speak for themselves:

|                 | Octopress   | Tinkerer     | Hugo      |
| --------------- | --------------- | --------------- | --------------- |
| Page generation | 7m35.910s   | 2m18.046s    | 15.123s           
| Site generation | 2m45.143s   | 59.87s       | 2.136s         


(The result may vary running the test in bare metal instead of in a -crappy- VM, but still they are very descriptives)

Based on the results:

* Ok, so Octopress is slow, and old... So what? It works really good out of the box, speed would be a problem if you post several times per day, but in my case, so it is not really an issue. For me the only thing that makes me think about changing the blog engine is Ruby... As simple as that
* Tinkerer looks like a neat product, and I will problably take another look at it, but since it doesn't allow markdown, I am not so sure if I feel like migrating the post to a different format... Still, it is based in Python, and it's faster than Octopress, so I will give it a try...
* Then should we move to Hugo blindfolded?? I don't think so... Not just yet. It is true that it is way faster, but I found it pretty complex for someone that doesn't really want to build the blog "almost" from scratch. Nevertheless, it looks really promising, it is surprisingly fast, and run in multiple OS (even from Windows)... So it might be a perfect alternative (at least for me) as soon as it is a little bit more mature.

Tricky decision...[Should I stay or should I go??](http://www.youtube.com/watch?v=GqH21LEmfbQ) :)

Bye!

