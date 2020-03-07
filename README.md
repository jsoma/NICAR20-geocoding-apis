# Geocoding API hands-on session

[Link to notebook](https://colab.research.google.com/github/jsoma/NICAR20-geocoding-apis/blob/master/Geocoding%20API%20worksheet.ipynb)

> Based on [Joe Fox's geocoding API](https://github.com/joemfox/NICAR20-geocoding) example

## What is geocoding?

You give me an address, I give you a latitude/longitude pair.

## Picking your geocoding option

* Dozens: Do it manually
* Hondreds-Thousands: [Geocod.io](http://geocod.io/) if you're US-only
* Tens of thousands, or international: See below! 
* Millions: do it yourself using Census TIGER data ([one](https://livebook.manning.com/book/postgis-in-action-second-edition/chapter-8/41), [two](http://web.mit.edu/11.188/www/lectures/lecture9.html)) if you're in the US

## Different tools

There are [many many geocoding APIs](https://github.com/DenisCarriere/geocoder#providers), all with different cover areas, costs, and APIs. To call out a few specifically:

* [Geocod.io](http://geocod.io/) isn't an API - you just upload a spreadsheet - and it's US/Canada only, but it's very easy to use.
* [Census Geocoder](https://geocoding.geo.census.gov/) is very slow and a little finnicky, but super free.
* [HERE Geocoding and Search](https://developer.here.com/documentation/geocoding-search-api/dev_guide/topics/endpoint-geocode-brief.html) is not to be confused with the old HERE Geocoding API, although _almost all documentation anywhere is for the old one_
* [Google Maps](https://developers.google.com/maps/documentation/geocoding/start) requires you to jump through a hundred hoops to get it to work, but no matter what kind of trash you might feed it it'll usually give you a pretty good answer. Unfortunately you should read [their policies](https://developers.google.com/maps/documentation/geocoding/policies), as they're pretty draconian. For example, you can't use Google's geocoder if you display the results on a non-Google map! I'm pretty sure no one follows that rule, but it's a great reason to try alternatives. 
* Once upon a time I used [Baidu's geocoder](http://lbsyun.baidu.com/index.php?title=webapi/guide/webservice-geocoding) and it was insanely blazing fast, but I definitely can't read the documentation so it isn't terribly useful for me.

The biggest issue is **how many free you get**. It changes over time, but it's typically 1000-2,5000. What if you have more than that? Usually people just spread it out over multiple days.

## Basic use

### Census Geocoder

From [the docs](https://geocoding.geo.census.gov/geocoder/Geocoding_Services_API.pdf):

```
https://geocoding.geo.census.gov/geocoder/locations/onelineaddress?address=4600+Silver+Hil
l+Rd%2C+Suitland%2C+MD+20746&benchmark=9&format=json
```

For **multiple addresses**, you need to send it a file! The file needs to be formatted a certain way (house number, street, etc, all split up). 

You can test this out with the command-line tool `curl` if you're exceptionally fancy:

```
curl --form addressFile=localfile.csv \
  https://geocoding.geo.census.gov/geocoder/locations/addressbatch \
  --output geocoded.csv
```

Using [geocoder](https://geocoder.readthedocs.io/):

```
> import geocoder
> g = geocoder.uscensus('455 Canal St, New Orleans, LA')
> g.ok
True
> g.latlng
[29.951624, -90.06652]
```

You can also use the [Geographies](https://geocoding.geo.census.gov/geocoder/geographies/onelineaddress?form) endpoint if instead of just the lat/lon you're interested in city/state/etc.

### HERE Geocoder

You'll need to [create a REST app](https://developer.here.com/documentation/authentication/dev_guide/topics/api-key-credentials.html) to create an API key.

```
https://geocode.search.hereapi.com/v1/geocode?q=5+Rue+Daunou%2C+75000+Paris%2C+France&apikey=YOUR_API_KEY
```

Unfortunately the [Geocoder library uses the old version](https://geocoder.readthedocs.io/providers/HERE.html) of HERE's geocoder.

### Google Maps

Using the raw API:

```
https://maps.googleapis.com/maps/api/geocode/json?address=1600+Amphitheatre+Parkway,
+Mountain+View,+CA&key=YOUR_API_KEY
```

Using the Geocoder library:

```
> import geocoder
> g = geocoder.google('455 Canal St, New Orleans, LA', key=YOUR_API_KEY)
> g.ok
True
> g.latlng
[29.951624, -90.06652]
```

## Gotchas

### What is an address?

You might want to read [falsehoods programmers believe about addresses](https://www.mjt.me.uk/posts/falsehoods-programmers-believe-about-addresses/), which has all sorts of weird addresses that you might encounter. It's more focused on how programmers like addresses to be _input_ into their systems, but there's a chance it'll apply to you as well.

### Formatting your data

Questions to ask and tests to run before you dedicate yourself to a geocoding solution:

* How far does the free tier take you?
* How well does it handle different abbreviations like N vs North? & vs AND?
* Does it only geocode addresses, or does it accept intersections? Names of things?
* Can you geocode an entire location at once, or do you need your data separated into separate fields? (address, stress and city in separate columns)
* How well does it handle misspellings?
* Usage restrictions (e.g. Google)

### Accuracy

The biggest issue with geocoding is **accuracy**. And I'm not talking, "oh the geocoder is inaccurate it doesn't know where anything is," I'm talking about **you being sure you know what level of accuracy you're getting.**

While we think of geocoding and latitude/longitude pairs as representing exact places on the surface of the earth, that isn't always the case with geocoding.

* If I geocode the United States, where will I end up? It won't say "that's too big of an area," it's probably going to put you somewhere in Kansas!
* If I geocode New Orleans, where will I end up? It probably won't pick a meaningful political or cultural center, instead you'll get the center of the city's boundaries.
* If it's a new address that seems to be between two existing addresses, where will the geocoder put you? It'll probably interpolate right in in the middle.
* If the geocoder can match the zip code, city, state or country, but **not the specific address**, what does it do? Is the zip code center good enough? The city's center? The state? 

For almost every geocoder, along with your latitude/longitude pair you get some measure of accuracy. **Pay attention to this!!!!!!!!** I can't tell you exactly what you're looking for, though: different geocoders report accuracy differently, and what you're _doing_ with the coordinates might have different requirements (e.g. are you mapping the points individually, or are you just counting the number of locations in each state?).

### Handling errors

### Add-ons

If you're into Python, [tqdm](https://github.com/tqdm/tqdm) is a fantastic progress bar library.

# Reverse geocoding

Tired of geocoding? There's also **reverse geocoding!** In the way that geocoding takes an address and gives you a lat/lon pair, for reverse geocoding you provide coordinates and receive location information instead.