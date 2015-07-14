title: Sharing images like a "Pr0"
slug: 2015-07-14-sharing-images-like-a-pro
date: 2015-07-14 13:00
category: posts
tag: images, internet, cli
status: draft

>> Disclaimer: it's summer, we are under a really long heat wave in Spain, so I didn't feel like moving from the couch (you will understand later)

Yesterday evening I was laying on the couch surfing Amazon website using an old laptop (very old) that I barely use, and at some point, I took an screenshot I needed to share with a friend via corporate email... (my idea was to send it to my email account, and this morning, send it from my corporate account)

Anyway, after taking the screenshot I realized:

  * I was no logged in gmail, dropbox, github, etc , and in order to login, I would need a token from my cellphone  (and my smartphone was in a different room!!!)
  * No USB drives around
  * The image sharing alternatives that I know of, requires you to create an user and  login… Too much for a onetime thing

So I decided to use [pastebin](http://pastebin.com/) to store the image in text format:

1. Take the screenshot (I used scrot, feel free to use any other tool…) : ```scrot -d 5 ~/amazon.jpg```

2. Gzip the file: ```gzip ~/amazon.jpg```

3. Convert to base64: ```base64 ~/amazon.jpg.gz > ~/amazon.b64```

4. Cat the file and paste the (long) result into pastebin and write down the  [url](http://pastebin.com/rgXncvEz9)

This morning from my corporate laptop:

5. Download the file from pastebin (save it as amazon.b64)

6. Set file format UNIX... For instance, using vi: ESC:set ff=unix

7. Decode it:```base64 -d amazon.b64 > amazon.jpg.gz```

8. Unzip it: ```gunzip amazon.jpg.gz```

Et voila, we have the same picture….

{%  img center https://dl.dropboxusercontent.com/u/14814182/blog/2015-07-14-Sharing-images-like-a-pro.jpg  amazon%}

This is not very useful (or yes, who knows?) but you might use the same approach for different things… Actually, I use pretty much the same thing in Python for embedding an old version of [pexpect]( https://github.com/pexpect/pexpect) module in my scripts

Later
