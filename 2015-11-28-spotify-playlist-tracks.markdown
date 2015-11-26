title: Spotify playlist tracks
date: 2015-11-28 11:28:15
slug: 2015-11-28-spotify-playlist-tracks
tag: spotify, python
category: posts
status: draft

I've never really understood why Spotify doesn't allow me to export my playlist into text file... I guess there is a reason (I bet they have one), but just don't see it...

Anyway, sometimes it's quicker to fix the problem yourself than go complain somewhere else.

Since Spotify has a nice API (with no auth needed) for [getting tracks information](https://developer.spotify.com/web-api/get-track/) available, I created a small script that would iterate over the songs in my playlist, and printout the information (Artist - Album - Song)

First of all, get the list of Spotify URIs from all the songs in your play list:

* CRL-A in your playlist.
* Click "Copy Spotify URI" from the context menu.
* Paste in a new file ie: "myplaylist.txt"

The file should look like this one:

```
$ cat myplaylist.txt
spotify:track:1OGFtaUgHAQjtSk7mhDwr9
spotify:track:39J10NL0mFTAdJbapoo2rC
spotify:track:3G0EKJZy0j3rMG077UawaC
spotify:track:5VdVaUBgj7cBTKplgaIhKu
spotify:track:2EBuFjexd3S3wc9m4Rerh8
```

See below the [script](https://raw.githubusercontent.com/psgonza/bynario/master/spotify_playlist.py):

```
#!/usr/bin/env python
# Getting track details from a list of Spotify URLs via spotify API
# Input file containing the list of URLs mandatory
# Date: 26 Nov 2015

import json, urllib2, sys
from time import sleep

#Checking input file
if len(sys.argv) != 2:
    print("Error: Input file mandatory")
    sys.exit(0)
else:
    input_file = sys.argv[1]

#Spotify track API
api_url = "https://api.spotify.com/v1/tracks/"

#track dictionary
mytracks = []

#Open input file
try:
    fd = open(input_file,"r")
except Exception as e:
    print("Error opening %s" % input_file)
    print(e)
    sys.exit(0)

#getting track details from uri
def get_track_data(url):

    request = urllib2.Request(api_url + url)
    request.add_header('User-Agent', 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)')
    request.add_header('Content-Type','application/json')

    try:
        response = urllib2.urlopen(request)
    except Exception as e:
        print("\nError connecting to spotify API looking details for uri %s" % url)
        return False

    try:
        data = json.load(response)
    except Exception as e:
        print("\nError handling json object looking details for uri %s" % url)
        return False

    try:
        mytracks.append({'album':str(data['album']['name']),'artist':str(data['artists'][0]['name']),'track':str(data['name'])})
        sys.stdout.write(".")
        sys.stdout.flush()
    except Exception as e:
        print("Error getting details for uri %s" % url)
        return False

########
# MAIN #
########

sys.stdout.write("Iterating through tracks in " + input_file)
sys.stdout.flush()

#Calling to get_track_data for each uri in the input file
for track in fd:
    get_track_data(track.split(":")[2])
    #I don't need to DDoS spotify servers :)
    sleep(2)
print("done!")

#Printing the header when the dict has been populated
print("\nArtist - Album - Track")
print("----------------------")

#Printing the results. By default, ordered by 'artist', but it can be ordered by 'album' or 'track'
for track in sorted(mytracks,key=lambda k: k['artist']):
    print("%s - %s - %s" % (track['artist'],track['album'],track['track']))

#close input file
fd.close()
```

A couple of things:

1) By default, the script is ordering the result by "artist", but it can be easily changed to "album" or "track" in this line:

```
for track in sorted(mytracks,key=lambda k: k['artist']):

--> for track in sorted(mytracks,key=lambda k: k['album']):

--> for track in sorted(mytracks,key=lambda k: k['track']):
```

2) I have decided to take it easy with Spotify servers, so I added `sleep 2` between each query.

3) No concurrency, multi-threading, etc... I don't plan to run the script on regular basis, so no need to spend more time on this.

Execution example:

```
$ python spotify_playlist.py myplaylist.txt
Iterating through tracks in myplaylist.txt......done!

Artist - Album - Track
----------------------
Adema - Adema - Giving In
Alice Cooper - The Best Of Alice Cooper - Poison
Alice In Chains - Facelift - Man in the Box
Alice In Chains - Jar Of Flies - Rotten Apple
Alice In Chains - Jar Of Flies - Nutshell
Alien Ant Farm - Anthology - Smooth Criminal
``` 

I know, I know... Only great hits in my [playlist](http://cdn.bynario.com/playlist.htm) :)
