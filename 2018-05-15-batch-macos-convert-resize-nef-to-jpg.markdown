Title: NEF batch processing in MacOS 
Date: 2018-05-15 00:05:15 +0200
Slug: 2018-05-15-batch-macos-convert-resize-nef-to-jpg
category: posts
tags: macos, scripting, note2self

Hi there

In case someone has to scratch the same itch as me (converting and resizing big NEF files to ease the visualization over the network), this is (by far, I'd say) the easiest way to generate (and resize) a jpg file from a big fat Nikon RAW file (NEF):

```
psgonza/Nikon$ for i in `ls -1 *.NEF`; 
do 
sips -s format jpeg -s formatOptions 50 -Z 3000 "${i}" --out "jpg/${i%NEF}jpg"; 
done
```

Above command will:
- Go through all the NEF files in current directory 
- Generate a jpg from the original nef file 
- Set jpg quality at 50% 
- Resize the image to max 3000px (either width or height, whatever is bigger) 

It took a WHILE to process all the files (in my case 1003 pictures), but I'd say it is worthy... I ended up dealing with (aprox) 1MB jpg files, instead of the original 25MB NEF files I had.

I am fairly new to the Mac OS world so I didn't know about ** sips ** until 30 min ago, although it has been included by default in Mac OS since 2003...

Pretty handy

