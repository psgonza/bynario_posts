title: Geolocating your holidays
date: 2016-12-20
slug: 2016-12-20-geolocating-your-holidays
tag: python, geotagging, google
category: draft

Tinkering with Python once again... Just for the sake of it :)

The idea is to generate a web page containing a Google Map with marks of your photo timestamps. I'm pretty sure Google Photos is able to do something similar (and better), but where's the fun in that? 

For instance, I spent a few days in Kauai this summer, and this is the resulting map:

{% img center https://dl.dropboxusercontent.com/u/14814182/blog/20-12-2016-geotagging_map.gif 'Kauai photos timestamp' %}

Each mark is a photo I took, and you can see its timestamp moving the mouse over the marker... 

Nothing fancy, but in case it is useful to you, this is the code:

```
#!/usr/bin/env python
"""
Giving a directory as input:
1) the script will extract the dates and GPS coordinates of the NEF files
2) will add a marker in a Map 
3) print the HTML file

Dependencies: 
- EXIF data has to contain GPS tags
- exifread module installed
- Google Map API credentials
"""
import exifread,sys,glob

G_API_KEY = '<GOOGLE_MAP_API_TOKEN>'

html="""<!DOCTYPE html>
<html>
<head>
    <meta name="viewport" content="initial-scale=1.0, user-scalable=no">
    <meta charset="utf-8">
    <title>Complex icons</title>
    <style>
      html, body {
        height: 100%;
        margin: 0;
        padding: 0;
      }
      #map {
        height: 100%;
      }
    </style>
  </head>
  <body>
    <div id="map"></div>
    <script>

function initMap() {
  var map = new google.maps.Map(document.getElementById('map'), {
    zoom: 10,
    center: {lat: xxxLATxxx, lng: xxxLONGxxx}
  });

  setMarkers(map);
}  

xxxCAPTURESxxx

function setMarkers(map) {
  for (var i = 0; i < captures.length; i++) {
    var capture = captures[i];
    var image = 'data:image/svg+xml,%3Csvg%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20width%3D%2238%22%20height%3D%2238%22%20viewBox%3D%220%200%2038%2038%22%3E%3Cpath%20fill%3D%22%23808080%22%20stroke%3D%22%23ccc%22%20stroke-width%3D%22.5%22%20d%3D%22M34.305%2016.234c0%208.83-15.148%2019.158-15.148%2019.158S3.507%2025.065%203.507%2016.1c0-8.505%206.894-14.304%2015.4-14.304%208.504%200%2015.398%205.933%2015.398%2014.438z%22%2F%3E%3Ctext%20transform%3D%22translate%2819%2018.5%29%22%20fill%3D%22%23fff%22%20style%3D%22font-family%3A%20Arial%2C%20sans-serif%3Bfont-weight%3Abold%3Btext-align%3Acenter%3B%22%20font-size%3D%2212%22%20text-anchor%3D%22middle%22%3E' + capture[3] + '%3C%2Ftext%3E%3C%2Fsvg%3E';
    var marker = new google.maps.Marker({
      position: { lat: capture[1], lng: capture[2] },
      map: map,
      icon: image,
      title: capture[0],
      zIndex: capture[3]
    });
  }
}
    </script>
    <script async defer
        src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&signed_in=true&callback=initMap"></script>
  </body>
</html> """ 

def formatme(mydata,reference):
    coordenates=(str(mydata).replace("[","").replace("]","").replace(" ","").split(","))

    d=coordenates[0]
    m=coordenates[1]
    s=coordenates[2]
    
    ref = str(reference).strip()

    d = int(d.split("/")[0]) / int(d.split("/")[1]) if "/" in d else int(d)
    m = int(m.split("/")[0]) / int(m.split("/")[1]) if "/" in m else int(m)
    s = int(s.split("/")[0]) / int(s.split("/")[1]) if "/" in s else int(s)

    coor = d + (m / 60.0) + (s / 3600.0)
    return coor if ref == "N" or ref == "E" else -coor

def find_pics(pics_path):
    #Some more logic could be added here
    files = glob.glob(pics_path + "/*.NEF") 
    files.sort(key=lambda x: os.path.getmtime(x))
    return files

def get_GPS_data(id,path):
    try:
        #Get the EXIF tags
        with open(path, 'rb') as f:
            tags = exifread.process_file(f,details=False)
    except Exception as e:
        data = ["skip"]

    #Check we got the GPS tag data
    if "GPS GPSLongitude" not in tags and "GPS GPSLongitudeRef" not in tags:
        data = ["skip"]
    else:
        data = [path,"{0}".format(tags["EXIF DateTimeOriginal"]),formatme(tags["GPS GPSLatitude"], \
                tags["GPS GPSLatitudeRef"]),formatme(tags["GPS GPSLongitude"], tags["GPS GPSLongitudeRef"]),id]

    return data

if __name__ == "__main__":
    try:
        path=sys.argv[1]
    except:
        print("Usage: " + sys.argv[0] + " <path>")
        sys.exit(1)

    #List of pictures to process
    pics_list=find_pics(path)

    #List of pics, dates, lat, long, id
    captures = []
    for _, pic in enumerate(pics_list):
        item = get_GPS_data(_, pic) 
        if "skip" not in item: captures.append(item)

    # Create the JS code with the list of coordenates
    html_loop = "var captures = ["
    for capture in captures:
        html_loop = html_loop + "".join("['" + str(capture[1]) + "'," + str(capture[2]) + "," + str(capture[3]) + \
                    "," + str(capture[4]) + "],\n")
    html_loop = html_loop[:-2] + "]; "
    
    #Update center LAT/LONG  coordenates
    html = html.replace("xxxLATxxx",str(captures[0][2]))
    html = html.replace("xxxLONGxxx",str(captures[0][3]))
    #Update list of points
    html = html.replace("xxxCAPTURESxxx",html_loop)
    #Update API KEY
    html = html.replace("YOUR_API_KEY",G_API_KEY)
    
    print(html)
```
[Stored in Github](https://github.com/psgonza/bynario/blob/master/GPS_tags_to_google_map.py)

The script is so simple that I am not even opening a file to write the output... Feel free to do it yourself, or redirect the output to a file:

``` # python extract_geolocation_batch_to_html.py > mymap.htm ```

Take care out there...

\\psgonza
