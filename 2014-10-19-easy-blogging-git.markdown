title: Easy blogging: Octopress, Git and Docker
date: 2014-11-02 10:00:02 +0200
slug: 2014-10-19-easy-blogging-git
comments: true
tags: Octopress,Blogging,Git,Docker

[Docker](https://www.docker.com) has been probably 2014 revelation (at least in IT world), so I have decided to get my hands on it, and at the same time, try to solve my "problem" with Ruby.

My idea is to "Dockerize" Octopress environment, and run it from a container every time I need to recreate the blog. And also, instead of keeping the blog posts (markdown files) locally, I will store them in a Git repository, and configure Git-hooks so the blog is recreated automatically.

First of all, make sure you have Docker and Git installed in your server.

In order to be able to build the blog, we will need [Octopress](http://octopress.org) source code. You can use your current Octopress installation or download it from Github:

```
$ git clone http://github.com/imathis/octopress.git octopress
$ cd octopress
```

Once Octopress is downloaded, let's create our Docker container using this [Dockerfile](http://docs.docker.com/reference/builder/):

```
FROM ubuntu:14.10

RUN apt-get update
RUN apt-get install git ruby1.9.3 ruby-dev build-essential language-pack-en nodejs python -y

ENV LC_ALL en_US.utf8

ADD . /blogSRC
WORKDIR /blogSRC

RUN adduser --uid 1000 --gid 50 deploy
RUN chown -R deploy:staff /blogSRC
USER deploy

RUN gem install bundler
RUN bundle install

ENTRYPOINT ["rake"]
```

This container will download and install Ruby environment and its dependences (There are some compatibility issues with Python3, so 2.7 has to be installed. See [this](https://github.com/imathis/octopress/issues/1651)), so we are able to run `rake` commands. 

Build the container:

```
$ sudo docker build -t bynario/octopress .
Sending build context to Docker daemon 3.829 MB
Sending build context to Docker daemon
Step 0 : FROM ubuntu:14.10
 ---> accae238329d
Step 1 : RUN apt-get update
 ---> Running in 5a456e103939
...
...
...
Step 11 : ENTRYPOINT ["rake"]
 ---> Running in c74572f9fb1a
 ---> 94d1b90e8e03
Removing intermediate container c74572f9fb1a
Successfully built 94d1b90e8e03
```

If it goes fine, you will see your new container by listing the images:

```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
bynario/octopress   latest              94d1b90e8e03        About an hour ago   531.8 MB
ubuntu              14.10               accae238329d        4 days ago          221.1 MB
$
```

If you downloaded Octopress from Github in Step 1, then we need to install it. Since we already have our container ready, let's use it:

```
$ sudo docker run --rm -v /home/docker/octopress/:/blogSRC bynario/octopress install
mkdir -p source
cp -r .themes/classic/source/. source
mkdir -p sass
cp -r .themes/classic/sass/. sass
mkdir -p source/_posts
mkdir -p public
## Copying classic theme into ./source and ./sass
$
```

(`/home/docker/octopress/` is my path to octopress installation folder, change it to fit your installation)

At this point our Octopress is propely installed, and Ruby is running from a Docker container, so there shouldn't be more issues with Ruby and system upgrades.

Now, let's get the posts from our git repository and configure the git-hook. For this I will assume that the posts are already commited and pushed to your git server (Github, BitBicker, etc), and your repositories are working just fine (ssh keys, users, etc..).

```
cd /home/docker/octopress/source/_posts
git init
git remote add origin git@server.domain:path/repository.git
git pull origin master
```

(Again, use the path that fits your installation, and update `git@server.domain:path/repository.git` with your own repository)

Now all the posts are in the `_posts` folder:

```
~/octopress/source/_posts$ ls -1
2011-01-01-tips-instalar-vmware-en-opensuse-11.markdown
...
...
2014-10-18-new-look-same-engine.markdown
2014-10-18-python-port-locker-slash-unlocker.markdown
~/octopress/source/_posts$
```

So we could generate the blog and check that it works. Let's run our docker container:

```
$ sudo docker run --rm -v /home/docker/octopress/:/blogSRC bynario/octopress generate
```

+ `docker run --rm`: run Docker and automatically clean up the container and remove the file system when the container exits.
+ `-v /home/docker/octopress/:/blogSRC`: Volume sharing. Mounting `/home/docker/octopress/` in the local machine as `/blogSRC` in the container.
+ `bynario/octopress`: Image name and tag.
+ `generate`: Rake command to be executed by the container.

And finally let's create our git-hook. There are (at least) two different ways of working:

+ Webhooks: The repository will communicate with a web server whenever the repository is pushed to.
+ Client-Side Hooks: Local scripts that are called when certain events occur.

I chose the latter. So I configured a `post-merge` hook, that will be executed every time the content of my `_post` directory is updated after running a `git pull`:

```
sudo docker run --rm -v /home/docker/octopress/:/blogSRC bynario/octopress generate
```

Don't forget the execution rights: `chmod +x /home/docker/octopress/source/_posts/.git/hooks/post-merge`

Now, it is up to you how to call the git-hook. For example, you could run it manually every time you post something new or schedule it every night in crontab:

```
* 00 * * *  cd /home/docker/octopress/source/_posts && /usr/bin/git pull origin master
```

Happy blogging!
