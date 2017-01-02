---
title: multiple return values and refactoring javascript
id: 103
date: 2010-04-28 15:57:44
tags:
- dynamic
- gmaps
- google maps
- javascript
- refactoring
---

I've been working on an entry for Victoria's AppMyState competition. Although I started by doing some javascript and some Ruby on Rails, the RoR part quickly became superfluous and I ended up writing a purely javascript app. It's a mash-up of Google Maps, Google's geocoding and hopefully some Street View too, so javascript is a good fit. There's also a python script to massage the government data in to a useable form, but that's behind the scenes.

The data I'm displaying is divided in to 11 categories and I want to treat each category separately, so I was ending up with repetitive code.

<pre lang="javascript">var horseTroughMgr = new MarkerManager(map);
var horseTroughMarkers = [];
var i = Horse_Trough.length;
while (i--) {
  var trough = Horse_Trough[i];
  var point = new GLatLng(trough.latitude, trough.longitude);
  var marker = createMarker(point,'

#### ' + trough.category + '
');
  horseTroughMarkers.push(marker);
  horseTroughMgr.addMarker(marker, 0);
}
horseTroughMgr.refresh();

var litterBinMgr = new MarkerManager(map);
var i = Litter_Bin.length;
while (i--) {
  var bin = Litter_Bin[i];
  var point = new GLatLng(bin.latitude, bin.longitude);
  var marker = createMarker(point,'

#### ' + bin.category + '
');
  litterBinMgr.addMarker(marker, 19);
}
litterBinMgr.refresh();

var hoopMgr = new MarkerManager(map);
var i = Hoop.length;
while (i--) {
  var hoop = Hoop[i];
  var point = new GLatLng(hoop.latitude, hoop.longitude);
  var marker = createMarker(point,'

#### ' + hoop.category + '
');
  hoopMgr.addMarker(marker, 15);
}
hoopMgr.refresh();</pre>

... and 8 more similar blocks.

The first block has an extra list of markers, and I wanted to add that to the other blocks. I also had an intuition that I didn't need both the list and the manager. I was holding back because I wasn't sure, and with the duplication I didn't feel that confident about refactoring. Then I remembered I could have two return values, and things started getting easy:

<pre lang="javascript">function setupCategory(map, data, minZoom) {
  var mgr = new MarkerManager(map);
  var list = [];

  var i = data.length;
  while (i--) {
    var item = data[i];
    var point = new GLatLng(item.latitude, item.longitude);
    var marker = createMarker(point,'

#### ' + item.category + '
');
    list.push(marker);
    mgr.addMarker(marker, minZoom);
  }
  mgr.refresh();
  return { manager: mgr, list: list };
}

function initialize() {
  var map = new GMap2(document.getElementById("map"));
  map.setCenter(new GLatLng(-37.8062649904, 144.96165842), 10);
  map.setUIToDefault();

  var trough = setupCategory(map, Horse_Trough, 0);
  var bin = setupCategory(map, Litter_Bin, 19);
  var hoop = setupCategory(map, Hoop, 15);</pre>

Having it in this form made it **very** easy to figure out that I didn't need the manager returned from setupCategory(), and I could only return the list. And because it was only in one place, it was easy to change.

This progression works well for me when I'm refactoring: eliminate the duplication, which makes it easier to see a way to simplify the code. Sometimes simplifying the code exposes more duplication and it turns in to a cycle, but not always. I'm often tempted to try to do this in two steps, but that usually ends up in trouble where I'm making mistakes, breaking things, and then hacking them back together.

Javascript's object literals made it easy to return two values which is what I needed here. In C# I would have needed a new class or some anonymous class and reflection. (Or maybe the dynamic keyword in C#4?) It is higher friction in static languages. This must be the reduced friction dynamic language enthusiasts brag about!

I should note that this seems not to be the best way to handle markers in many categories. It's better to have a manager per-category.
