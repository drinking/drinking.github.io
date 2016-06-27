---
layout: page
title: Tour
geomap: true
permalink: /tour/
---

This will be a guideline to  lead a great journey to Tiet.
I think we should do something special when we are young.
For more photos about my traveling please follow me on Instagram [@duyado](https://www.instagram.com/duyado/).

<link rel="stylesheet" href="http://cdn.leafletjs.com/leaflet/v0.7.7/leaflet.css" />
<link rel="stylesheet" href="{{site.url}}/assets/routing/leaflet-routing-machine.css" />
<script src="http://cdn.leafletjs.com/leaflet/v0.7.7/leaflet.js"></script>
<script src="{{site.url}}/assets/routing/leaflet-routing-machine.js"></script>


<style type="text/css">
     #map {
     width:100%;
     height:400px;
     background-color: white;
   }
  </style>


<div id="map" class="map"></div>
<script>
var map = L.map('map');

L.tileLayer('http://{s}.tile.osm.org/{z}/{x}/{y}.png', {
    attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
}).addTo(map);

    L.Routing.control({
  waypoints: [
    L.latLng(31.211480,121.634960),
    L.latLng(30.564725,104.073796),
    L.latLng(29.652020,91.163766)
  ]
}).addTo(map).hide(); //hide navigation bar


function createLink(label, container) {
    var btn = L.DomUtil.create('a', '', container);
    btn.setAttribute('href', '');
    btn.innerHTML = label;
    return btn;
}

map.on('click', function(e) {
    var container = L.DomUtil.create('div'),
    destBtn = createLink('Preparing', container);
    L.popup().setContent(container)
        .setLatLng(e.latlng)
        .openOn(map);
});
</script>

