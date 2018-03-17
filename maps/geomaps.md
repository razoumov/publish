## Goals

- must be presented interactively in a web browser
- must have a zoomable/scrollable 2D heatmap of a continuous variable such air quality, pressure,
  etc. plotted over a recognizable geographic map
  - ideally the heatmap would be encoded in the tiles at any resolution level like in
    https://maps.darksky.net
  - would be nice to be able to switch between fields
- must show individual weather stations, and a mouse-over should open more data from that station

## Tools

### basemap matplotlib toolkit
- [extension to matplotlib](https://matplotlib.org/basemap/users/examples.html): not interactive, creates
  a static PNG
- this is what they already use -- see FireWork_ModelPerfAtStn.png
- results can be [really nice](https://python-graph-gallery.com/315-a-world-map-of-surf-tweets)

### Follium
- [Folium Python library](https://folium.readthedocs.io/en/latest) visualizes data on an interactive Leaflet.js map
- docs: [Folium Quickstart Guide](https://folium.readthedocs.io/en/latest/quickstart.html),
  [official documentation](https://folium.readthedocs.io/en/latest),
  [useful script examples](https://github.com/python-visualization/folium/tree/master/examples),
  install with *pip install folium*
- what they call [*heatmaps*](http://qingkaikong.blogspot.com/2016/06/using-folium-3-heatmap.html) do
  not show a continuous distribution but rather frequency/location of discrete events or map locations
  that blend into a
  [frequency heatmap](https://cdn.rawgit.com/razoumov/publish/058067cd/maps/foliumHeatMap.html) (with
  foliumHeatMap.py) when you zoom out (think earthquakes, bus stops, schools, etc)
- can use [a mesh of markers](https://cdn.rawgit.com/razoumov/publish/60563ccd/maps/foliumCircles.html)
  (with foliumCircles.py)

### plot.ly
- docs: [tutorials](https://plot.ly/python), [examples](https://plot.ly/create),
  [keyword index](https://plot.ly/python/reference), [getting started guide](https://plot.ly/python/getting-started)
- free, open-source, does not require an online plot.ly account
- [community feed with gallery](https://plot.ly/feed)
- [regenerate your API key](https://plot.ly/settings/api) for online plotting
- really great with
  [markers on geographical maps](https://cdn.rawgit.com/razoumov/publish/845eaae6/maps/citiesByPopulation.html)
  (with citiesByPopulation.py)
- plot.ly does not support geographical heatmaps or filled contours or even shapes that are tied to the
  map; as a workaround, one could show geo-spatial distribution on top of maps with:
  1. choropleths over custom areas (really a hack, fairly difficult to do)
  1. scatter plots, e.g.,
     [markers on a regular grid](https://cdn.rawgit.com/razoumov/publish/cedeb718/maps/geoMesh.html)
     (with geoMesh.py) or
     [even combining markers with choropleth maps](https://cdn.rawgit.com/razoumov/publish/21b20d58/maps/combine.html)
     (with combine.py), but cannot combine with heatmaps (one will overwrite the other)
	 1. lines showing contours (need to be computed separately), but NOT filled contours

### bokeh
- what they call *interactive maps* is just a non-rectangular heatmap (with zoom/pan), no underlying
  geographical map; e.g., see the bottom image in http://bit.ly/2G1PzZT
