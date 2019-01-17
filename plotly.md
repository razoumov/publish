<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Plot.ly for scientific visualization](#plotly-for-scientific-visualization)
  - [Installation](#installation)
  - [Online documentation:](#online-documentation)
  - [Initial setup for online plotting](#initial-setup-for-online-plotting)
- [Three different plot destinations](#three-different-plot-destinations)
  - [Online plotting](#online-plotting)
  - [Offline plotting](#offline-plotting)
  - [Plotting inside a Jupyter notebook](#plotting-inside-a-jupyter-notebook)
- [2D plots: deeper dive](#2d-plots-deeper-dive)
  - [Scatter plots](#scatter-plots)
    - [Bar plots](#bar-plots)
    - [Heatmaps](#heatmaps)
    - [Contour maps](#contour-maps)
    - [Geographical scatterplot](#geographical-scatterplot)
  - [3D plots](#3d-plots)
    - [Topographic elevation](#topographic-elevation)
    - [Elevated 2D functions](#elevated-2d-functions)
    - [Lighting control](#lighting-control)
    - [Parametric plots](#parametric-plots)
    - [Scatter plots](#scatter-plots-1)
    - [Graphs](#graphs)
    - [3D functions](#3d-functions)
    - [NetCDF data](#netcdf-data)
      - [2D slices through a 3D dataset](#2d-slices-through-a-3d-dataset)
      - [Orthogonal 2D slices](#orthogonal-2d-slices)
  - [Animation](#animation)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

You can find these notes at http://bit.ly/plotlymd and the CSV/NetCDF data files at
https://transfer.sh/98Bt5/csv.zip (ZIP archive). Solutions to exercises are in `solutions.md` on
presenter's laptop.

# Plot.ly for scientific visualization

* developed by a Montreal-based company <a href="https://plot.ly" target="_blank">Plot.ly</a>
* open-source scientific plotting Python library for Python, R, MATLAB, Perl, Julia
* front end uses JavaScript/HTML/CSS and D3.js visualization library
* files are hosted on Amazon S3

Generated plots can be
1. browsed online
1. stored as offline html5 files
1. displayed inside a Jupyter notebook

## Installation

- Python 3, along with the *pip* package manager
- run the command
~~~ {.bash}
$ pip install plotly
~~~
- optionally sign up for a free Plot.ly account at https://plot.ly/accounts/login/?action=signup
- optionally install *jupyter*

## Online documentation:

1. [Tutorials](https://plot.ly/python)
1. [Code examples](https://plot.ly/create)
1. [Keyword index](https://plot.ly/python/reference)
1. [Getting started guide](https://plot.ly/python/getting-started)
1. [Community feed with gallery](https://plot.ly/feed)
1. [Plotly for matplotlib users](https://plot.ly/matplotlib/getting-started)
1. [Using Plotly with Python offline](https://plot.ly/python/offline)
1. [Saving static images (PNG, PDF, etc) ](https://plot.ly/python/static-image-export)
1. [Creating HTML or PDF reports in Python](https://plot.ly/python/#report-generation)
1. [Creating dashboards with Plotly and Python](https://plot.ly/python/dashboard)
1. [Connecting to databases](https://plot.ly/python/#databases)
1. [Plotly and IPython / Jupyter notebook](https://plot.ly/ipython-notebooks)

## Initial setup for online plotting

Setting up online plotting (need to do this only once):
1. obtain a free account at https://plot.ly/accounts/login/?action=signup
1. generate [your API key](https://plot.ly/settings/api)
1. run the following from Python
~~~ {.python}
import plotly
plotly.tools.set_credentials_file(username='yourUserName', api_key='yourAPIkey')   # will write to ~/.plotly/.credentials
~~~

# Three different plot destinations

## Online plotting

* room to store 25 free charts; beyond that will get an error
* can free up some space by removing older plots using their Python web API or simply deleting individual
  plots at https://plot.ly/organize/home

~~~ {.python}
import plotly.plotly as py    # online plotting
import plotly.graph_objs as go
from numpy import linspace, sin
x1 = linspace(0.01,1,100)
y1 = sin(1/x1)
trace1 = go.Scatter(x=x1, y=y1, mode='lines+markers', name='sin(1/x)')
data = [trace1]
py.plot(data,auto_open=False)   # create a unique URL for this plot and optionally open it
~~~

This should return the URL of the plot and open it in your web browser (no local plots saved).

~~~ {.python}
help(py.plot) to show all plotting arguments
py.plot(data, auto_open=False)   # make the online plot, do not auto-open in the web browser
py.plot(data, auto_open=False, sharing='private')   # only 1 free private file
~~~

With default one free _private_ file, you will quickly reach the quota. Same for _secret_ keyword (need a
paid account). However, you get 25 free public plots, and unlimited offline plotting!

## Offline plotting

1. change the first line,
1. specify filename in last line.

~~~ {.python}
import plotly.offline as py   # offline plotting
import plotly.graph_objs as go
from numpy import linspace, sin
x1 = linspace(0.01,1,100)
y1 = sin(1/x1)
trace1 = go.Scatter(x=x1, y=y1, mode='lines+markers', name='sin(1/x)')
data = [trace1]
py.plot(data, filename='lines.html',auto_open=False)
~~~

By default this will auto-open the file, but you can also use `auto_open=False` if you want.

## Plotting inside a Jupyter notebook

~~~ {.bash}
$ jupyter notebook
~~~

1. start a new Python 3 notebook
1. in the first line switch back to plotly.plotly
1. in the last line change py.plot() to py.iplot()

~~~ {.python}
import plotly.plotly as py
import plotly.graph_objs as go
from numpy import linspace, sin
x1 = linspace(0.01,1,100)
y1 = sin(1/x1)
trace1 = go.Scatter(x=x1, y=y1, mode='lines+markers', name='sin(1/x)')
data = [trace1]
py.iplot(data)   # create a unique URL and open the plot inline in a Jupyter Notebook
~~~

For this one, an online plot will also be created -- you can see the URL by clicking `Edit Chart` button.

For offline plotting in Jupyter, you need to use:

~~~ {.python}
import plotly.plotly as py
py.init_notebook_mode(connected=True)
...
py.iplot(data)
~~~

* `connected=True` will use the online plotly.js library inside the notebook (smaller notebook file sizes)
* `connected=False` will include plotly.js into the notebook for complete offline work (larger files
  sizes, also need to modify some notebook settings before it works)

# 2D plots: deeper dive
## Scatter plots

Let's go back to the offline version of the code:

~~~ {.python}
import plotly.plotly as py   # offline plotting
import plotly.graph_objs as go
from numpy import linspace, sin
x1 = linspace(0.01,1,100)
y1 = sin(1/x1)
trace1 = go.Scatter(x=x1, y=y1, mode='lines+markers', name='sin(1/x)')   # dataset
data = [trace1]     # a list of datasets
py.plot(data,auto_open=False)
~~~

Let's print the dataset `trace1`: it is a plotly object which is actually a Python dictionary, with all
elements clearly identified (plot type, x numpy array, y numpy array, line type, legend line name). So,
`go.Scatter` simply creates a dictionary with the corresponding `type` element. **This variable/dataset
`trace1` completely describes our plot!*** Then we create a list `data` of such objects and pass it to
the plotting routine.

In fact, we can rewrite this routine with dictionaries:

~~~ {.python}
import plotly.plotly as py   # offline plotting
from numpy import linspace, sin
x1 = linspace(0.01,1,100)
y1 = sin(1/x1)
trace1 = dict(type='scatter', x=x1, y=y1, mode='lines+markers', name='sin(1/x)')
data = [trace1]
py.plot(data,auto_open=False)
~~~

> ## Exercise 1
> Pass a list of two objects the plotting routine with `data = [trace1,trace2]`. Let the second dataset
> `trace2` contain another mathematical function. The idea is to have multiple objects in the plot.

Notice:
* how we can hover over each data point, and its (x,y) will be shown
* the toolbar at the top
* double-clicking on the plot will reset it

> ## Exercise 2
> Add a bunch of dots to the plot with `dots = go.Scatter(x=[.2,.4,.6,.8], y=[2,1.5,2,1.2])`. What is
> default scatter mode?

> ## Exercise 3
> Change line colour and width by adding the dictionary `line=dict(color=('rgb(205,12,24)'),width=4)` to
> `dots`:

What are the plot types? There are quite a few:

~~~ {.python}
import plotly.graph_objs as go
dir(go)
~~~
~~~
['AngularAxis', 'Annotation', 'Annotations', 'Area', 'Bar', 'Box', 'Candlestick', 'Carpet', 'Choropleth', 'ColorBar', 'Contour', 'Contourcarpet', 'Contours', 'Data', 'ErrorX', 'ErrorY', 'ErrorZ', 'Figure', 'Font', 'Frames', 'Heatmap', 'Heatmapgl', 'Histogram', 'Histogram2d', 'Histogram2dContour', 'Histogram2dcontour', 'Layout', 'Legend', 'Line', 'Margin', 'Marker', 'Mesh3d', 'Ohlc', 'Parcoords', 'Pie', 'Pointcloud', 'RadialAxis', 'Sankey', 'Scatter', 'Scatter3d', 'Scattercarpet', 'Scattergeo', 'Scattergl', 'Scattermapbox', 'Scatterternary', 'Scene', 'Stream', 'Surface', 'Table', 'Trace', 'XAxis', 'XBins', 'YAxis', 'YBins', 'ZAxis', '__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__path__', '__spec__', 'absolute_import', 'graph_objs', 'graph_objs_tools']
~~~

### Bar plots

Let's try a Bar plot, constructing `data` directly in one line from the dictionary:

~~~ {.python}
import plotly.plotly as py
import plotly.graph_objs as go
data = [go.Bar(x=['Vancouver', 'Calgary', 'Toronto', 'Montreal', 'Halifax'],
               y=[2463431, 1392609, 5928040, 4098927, 403131])]
py.plot(data,auto_open=False)
~~~

Let's plot inner city population vs. greater metro area for each city:

~~~ {.python}
import plotly.plotly as py
import plotly.graph_objs as go
cities = ['Vancouver', 'Calgary', 'Toronto', 'Montreal', 'Halifax']
proper = [631486, 1239220, 2731571, 1704694, 316701]
metro = [2463431, 1392609, 5928040, 4098927, 403131]
bar1 = go.Bar(x=cities, y=proper, name='inner city')
bar2 = go.Bar(x=cities, y=metro, name='greater area')
data = [bar1,bar2]
py.plot(data,auto_open=False)   # we get a grouped bar chart
~~~

Let's now do a stacked plot, with *outer city* population on top of *inner city* population:

~~~ {.python}
import plotly.plotly as py
import plotly.graph_objs as go
cities = ['Vancouver', 'Calgary', 'Toronto', 'Montreal', 'Halifax']
proper = [631486, 1239220, 2731571, 1704694, 316701]
metro = [2463431, 1392609, 5928040, 4098927, 403131]
outside = [m-p for p,m in zip(proper,metro)]   # need to subtract
bar1 = go.Bar(x=cities, y=proper, name='inner city')
bar2 = go.Bar(x=cities, y=outside, name='outer city')
data = [bar1,bar2]
layout = go.Layout(barmode='stack')         # new element!
fig = go.Figure(data=data, layout=layout)   # new element!
py.plot(fig,auto_open=False)   # we get a stacked bar chart
~~~

What else can we modify in the layout?

~~~ {.python}
import plotly.graph_objs as go
help(go.Layout)
~~~

There are lots of attributes! Let's set the **title** and the **background colour**:

~~~ {.python}
layout = go.Layout(barmode='stack', title='Population', plot_bgcolor = 'rgb(153, 204, 255)')
~~~

### Heatmaps

* go.Area() for plotting **wind rose charts**
* go.Box() for **basic box plots**

Let's plot a heatmap of monthly temperatures at the South Pole:

~~~ {.python}
import plotly.plotly as py
import plotly.graph_objs as go
months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec', 'Year']
recordHigh = [-14.4,-20.6,-26.7,-27.8,-25.1,-28.8,-33.9,-32.8,-29.3,-25.1,-18.9,-12.3,-12.3]
averageHigh = [-26.0,-37.9,-49.6,-53.0,-53.6,-54.5,-55.2,-54.9,-54.4,-48.4,-36.2,-26.3,-45.8]
dailyMean = [-28.4,-40.9,-53.7,-57.8,-58.0,-58.9,-59.8,-59.7,-59.1,-51.6,-38.2,-28.0,-49.5]
averageLow = [-29.6,-43.1,-56.8,-60.9,-61.5,-62.8,-63.4,-63.2,-61.7,-54.3,-40.1,-29.1,-52.2]
recordLow = [-41.1,-58.9,-71.1,-75.0,-78.3,-82.8,-80.6,-79.3,-79.4,-72.0,-55.0,-41.1,-82.8]
trace = go.Heatmap(z=[recordHigh, averageHigh, dailyMean, averageLow, recordLow],
                   x=months,
                   y=['record high', 'aver.high', 'daily mean', 'aver.low', 'record low'])
data = [trace]
py.plot(data,auto_open=False)
~~~

### Contour maps

> ## Exercise 4
> Pretend that our heatmap is defined over a 2D domain and plot the same temperature data as a contour
> map. Remove the `Year` data (last column) and use `go.Contour` to plot the 2D contour map.

Let's change to a different colourmap:

~~~
11c11
<                    y=['record high', 'aver.high', 'daily mean', 'aver.low', 'record low'])
---
>                    y=['record high', 'aver.high', 'daily mean', 'aver.low', 'record low'],
>                    colorscale='Jet')
~~~

### Geographical scatterplot

Now let's do a scatterplot on top of a geographical map:

~~~ {.python}
import plotly.plotly as py
import plotly.graph_objs as go
import pandas as pd
from math import log10
df = pd.read_csv('cities.csv')   # lists name,pop,lat,lon for 254 Canadian cities and towns
df['text'] = df['name'] + '<br>Population ' + \
             (df['pop']/1e6).astype(str) +' million' # add new column for mouse-over
largest, smallest = df['pop'].max(), df['pop'].min()
def normalize(x):
    return log10(x/smallest)/log10(largest/smallest)   # x scaled into [0,1]
df['logsize'] = round(df['pop'].apply(normalize)*255)   # new column
cities = go.Scattergeo(
    lon = df['lon'], lat = df['lat'], text = df['text'],
    marker = dict(
        size = df['pop']/5000,
        color = df['logsize'],
        colorscale = 'Viridis',
        showscale = True,   # show the colourbar
        line = dict(width=0.5, color='rgb(40,40,40)'),
        sizemode = 'area'))
layout = go.Layout(title = 'City populations',
                       showlegend = False,   # do not show legend for first plot
                       geo = dict(
                           scope = 'north america',
                           resolution = 50,   # base layer resolution of km/mm
                           lonaxis = dict(range=[-130,-55]), lataxis = dict(range=[44,70]), # plot range
                           showland = True, landcolor = 'rgb(217,217,217)',
                           showrivers = True, rivercolor = 'rgb(153,204,255)',
                           showlakes = True, lakecolor = 'rgb(153,204,255)',
                           subunitwidth = 1, subunitcolor = "rgb(255,255,255)",   # province border
						   countrywidth = 2, countrycolor = "rgb(255,255,255)"))  # country border
fig = go.Figure(data=[cities], layout=layout)
py.plot(fig,auto_open=False)
~~~

> ## Exercise 5
> Modify the code to display only 10 largest cities.

Recall how we combined several scatter plots in one figure before. You can combine several plots on top
of a single map -- let's **combine scattergeo + choropleth**:

~~~ {.python}
import plotly.plotly as py
import plotly.graph_objs as go
import pandas as pd
df = pd.read_csv('cities.csv')
df['text'] = df['name'] + '<br>Population ' + \
             (df['pop']/1e6).astype(str)+' million' # add new column for mouse-over
cities = go.Scattergeo(lon = df['lon'],
                       lat = df['lat'],
                       text = df['text'],
                       marker = dict(
                           size = df['pop']/5000,
                           color = "lightblue",
                           line = dict(width=0.5, color='rgb(40,40,40)'),
                           sizemode = 'area'))
gdp = pd.read_csv('gdp.csv')   # read name, gdp, code for 222 countries
c1 = [0,"rgb(5, 10, 172)"]     # define colourbar from top (0) to bottom (1)
c2, c3 = [0.35,"rgb(40, 60, 190)"], [0.5,"rgb(70, 100, 245)"]
c4, c5 = [0.6,"rgb(90, 120, 245)"], [0.7,"rgb(106, 137, 247)"]
c6 = [1,"rgb(220, 220, 220)"]
countries = go.Choropleth(locations = gdp['CODE'],
                          z = gdp['GDP (BILLIONS)'],
                          text = gdp['COUNTRY'],
                          colorscale = [c1,c2,c3,c4,c5,c6],
                          autocolorscale = False,
                          reversescale = True,
                          marker = dict(line = dict(color='rgb(180,180,180)',width = 0.5)),
                          zmin = 0,
                          colorbar = dict(tickprefix = '$',title = 'GDP<br>Billions US$'))
layout = go.Layout(hovermode = "x", showlegend = False)  # do not show legend for first plot
fig = go.Figure(data=[cities,countries], layout=layout)
py.plot(fig,auto_open=False)
~~~

## 3D plots

### Topographic elevation

Let's plot some tabulated topographic elevation data:

~~~ {.python}
import plotly.plotly as py
import plotly.graph_objs as go
import pandas as pd
table = pd.read_csv('mt_bruno_elevation.csv')
data = go.Surface(z=table.as_matrix())  # use 2D numpy array format
layout = go.Layout(title='Mt Bruno Elevation',
                   width=800, height=800,    # image size
                   margin=dict(l=65, r=10, b=65, t=90))   # margins around the plot
fig = go.Figure(data=[data], layout=layout)
py.plot(fig,auto_open=False)
~~~

### Elevated 2D functions

> ## Exercise 6
> Plot a 2D function f(x,y) = (1−y) sin(πx) + y sin^2(2πx), where x,y ∈ [0,1] on a 100^2 grid.

Let's define a different colourmap by adding `colorscale='Viridis'` inside `go.Surface()`. This is our
current code:

~~~ {.python}
import plotly.plotly as py
import plotly.graph_objs as go
from numpy import *
n = 100   # plot resolution
x = linspace(0,1,n)
y = linspace(0,1,n)
Y, X = meshgrid(x, y)   # meshgrid() returns two 2D arrays storing x/y respectively at each mesh point
F = (1-Y)*sin(pi*X) + Y*(sin(2*pi*X))**2   # array operation
data = go.Surface(z=F, colorscale='Viridis')
layout = go.Layout(width=1000, height=1000, scene=go.Scene(zaxis=go.ZAxis(range=[-1,2])));
fig = go.Figure(data=[data], layout=layout)
py.plot(fig,auto_open=False)
~~~

### Lighting control

Let's change the default light in the room by adding `lighting=dict(ambient=0.1)` inside
`go.Surface()`. Now our plot is much darker!

* `ambient` controls the light in the room (default = 0.8)
* `roughness` controls amount of light scattered (default = 0.5)
* `diffuse` controls the reflection angle width (default = 0.8)
* `fresnel` controls light washout (default = 0.2)
* `specular` induces bright spots (default = 0.05)

Let's try `lighting=dict(ambient=0.1,specular=0.3)` -- now we have lots of specular light!

### Parametric plots

In plotly documentation you can find quite a lot of
[different 3D plot types](https://plot.ly/python/3d-charts). Here is something visually very different,
but it still uses `go.Surface(x,y,z)`:

~~~ {.python}
import plotly.plotly as py
import plotly.graph_objs as go
from numpy import pi, sin, cos, mgrid
dphi, dtheta = pi/250, pi/250    # 0.72 degrees
[phi, theta] = mgrid[0:pi+dphi*1.5:dphi, 0:2*pi+dtheta*1.5:dtheta]
        # define two 2D grids: both phi and theta are (252,502) numpy arrays
r = sin(4*phi)**3 + cos(2*phi)**3 + sin(6*theta)**2 + cos(6*theta)**4
x = r*sin(phi)*cos(theta)   # x is also (252,502)
y = r*cos(phi)              # y is also (252,502)
z = r*sin(phi)*sin(theta)   # z is also (252,502)
surface = go.Surface(x=x, y=y, z=z, colorscale='Viridis')
layout = go.Layout(title='parametric plot')
fig = go.Figure(data=[surface], layout=layout)
py.plot(fig,auto_open=False)
~~~

### Scatter plots

Let's take a look at a 3D scatter plot using the `country index` data from http://www.prosperity.com for
for 142 countries:

~~~ {.python}
import plotly.plotly as py
import plotly.graph_objs as go
import pandas as pd
df = pd.read_csv('legatum2015.csv')
spheres = go.Scatter3d(x=df.economy,
                       y=df.entrepreneurshipOpportunity,
                       z=df.governance,
                       text=df.country,
                       mode='markers',
                       marker=dict(
                           sizemode = 'diameter',
                           sizeref = 0.3,   # max(safetySecurity+5.5) / 32
                           size = df.safetySecurity+5.5,
                           color = df.education,
                           colorscale = 'Viridis',
                           colorbar = dict(title = 'Education'),
                           line = dict(color='rgb(140, 140, 170)')))   # sphere edge
layout = go.Layout(height=900, width=900,
                   title='Each sphere is a country sized by safetySecurity',
                   scene = dict(xaxis=dict(title='economy'),
                                yaxis=dict(title='entrepreneurshipOpportunity'),
                                zaxis=dict(title='governance')))
fig = go.Figure(data=[spheres], layout=layout)
py.plot(fig,auto_open=False)
~~~

### Graphs

We can plot 3D graphs. Consider a Dorogovtsev-Goltsev-Mendes graph: *in each subsequent generation, every
edge from the previous generation yields a new node, and the new graph can be made by connecting together
three previous-generation graphs*.

~~~ {.python}
import plotly.plotly as py
import plotly.graph_objs as go
import networkx as nx
from forceatlas import forceatlas2_layout
import sys
generation = int(sys.argv[1])
H = nx.dorogovtsev_goltsev_mendes_graph(generation)
print(H.number_of_nodes(), 'nodes and', H.number_of_edges(), 'edges')
# Force Atlas 2 graph layout from https://github.com/tpoisot/nxfa2.git
pos = forceatlas2_layout(H, iterations=100, kr=0.001, dim=3)
Xn = [pos[i][0] for i in pos]   # x-coordinates of all nodes
Yn = [pos[i][1] for i in pos]   # y-coordinates of all nodes
Zn = [pos[i][2] for i in pos]   # z-coordinates of all nodes
Xe, Ye, Ze = [], [], []
for edge in H.edges():
    Xe += [pos[edge[0]][0], pos[edge[1]][0], None]   # x-coordinates of all edge ends
    Ye += [pos[edge[0]][1], pos[edge[1]][1], None]   # y-coordinates of all edge ends
    Ze += [pos[edge[0]][2], pos[edge[1]][2], None]   # z-coordinates of all edge ends
degree = [deg[1] for deg in H.degree()]   # list of degrees of all nodes
labels = [str(i) for i in range(H.number_of_nodes())]
edges = go.Scatter3d(x=Xe, y=Ye, z=Ze,
                     mode='lines',
                     line=go.Line(color='rgb(160,160,160)', width=2),
                     hoverinfo='none')
nodes = go.Scatter3d(x=Xn, y=Yn, z=Zn,
                     mode='markers',
                     marker=go.Marker(sizemode = 'area',
                                      sizeref = 0.01, size=degree,
                                      color=degree, colorscale='Viridis',
                                      line=go.Line(color='rgb(50,50,50)', width=0.5)),
                     text=labels, hoverinfo='text')
axis = dict(showbackground=False, showline=False, zeroline=False, showgrid=False, showticklabels=False, title='')
layout = go.Layout(
    title = str(generation) + "-generation Dorogovtsev-Goltsev-Mendes graph",
    width=1000, height=1000,
    showlegend=False,
    scene=go.Scene(xaxis=go.XAxis(axis),
                   yaxis=go.YAxis(axis),
                   zaxis=go.ZAxis(axis)),
    margin=go.Margin(t=100))
fig = go.Figure(data=[edges,nodes], layout=layout)
py.plot(fig,auto_open=False)
~~~

~~~ {.bash}
$ python test.py 2
$ python test.py 3
$ python test.py 4
~~~

### 3D functions

Let's create an isosurface of a `decoCube` function at f=0.03. Isosurfaces are returned as a list of
polygons, and for plotting polygons in plotly we need to use `plotly.figure_factory.create_trisurf()`
which replaces `plotly.graph_objs.Figure()`:

~~~ {.python}
import plotly.plotly as py
from plotly import figure_factory as FF
from numpy import mgrid
from skimage import measure
X,Y,Z = mgrid[-1.2:1.2:30j, -1.2:1.2:30j, -1.2:1.2:30j] # three 30^3 grids, each side [-1.2,1.2] in 30 steps
F = ((X*X+Y*Y-0.64)**2 + (Z*Z-1)**2) * \
    ((Y*Y+Z*Z-0.64)**2 + (X*X-1)**2) * \
    ((Z*Z+X*X-0.64)**2 + (Y*Y-1)**2)
vertices, triangles, normals, values = measure.marching_cubes(F, 0.03)  # create an isosurface
x,y,z = zip(*vertices)   # zip(*...) is opposite of zip(...): unzips a list of tuples
fig = FF.create_trisurf(x=x, y=y, z=z, plot_edges=False,
                        simplices=triangles, title="Isosurface", height=900, width=900)
py.plot(fig,auto_open=False)
~~~

Try switching `plot_edges=False` to `plot_edges=True` -- you'll see individual polygons!

### NetCDF data
#### 2D slices through a 3D dataset

How about processing real data?

~~~ {.python}
import plotly.plotly as py
import plotly.graph_objs as go
from netCDF4 import Dataset
dataset = Dataset('sineEnvelope.nc')
name = list(dataset.variables.keys())[0]   # variable name
print(name)
var = dataset.variables[name]
print(var.shape)
image = go.Heatmap(z=var[:,:,49])   # use layer 50 (in the middle)
layout = go.Layout(width=800, height=800, margin=dict(l=65,r=10,b=65,t=90))
fig = go.Figure(data=[image], layout=layout)
py.plot(fig,auto_open=False)
~~~

> ## Exercise 7
> Use the last two codes to plot an isosurface of `sineEnvelope.nc` dataset at f=0.5.

#### Orthogonal 2D slices

Let's do three orthogonal slices through our dataset:

~~~ {.python}
import numpy as np
import plotly.plotly as py
import plotly.graph_objs as go
from netCDF4 import Dataset
dataset = Dataset('sineEnvelope.nc')
name = list(dataset.variables.keys())[0]   # variable name
var = dataset.variables[name]   # dataset variable
vmin, vmax = np.min(var), np.max(var)   # global dataset min/max
x, y, z = np.linspace(0.005,0.995,100), np.linspace(0.005,0.995,100), np.linspace(0.005,0.995,100)
slicePosition = [0,0,0]   # indices 0..99
print('slices through the point', x[slicePosition[0]], y[slicePosition[1]], z[slicePosition[2]])

# --- create the XY-slice
X, Y = np.meshgrid(x,y)   # each is a 100x100 mesh covering [0,1] in each dimension
Z = z[slicePosition[2]] * np.ones((100,100))   # 100x100 array of constant value
surfz = var[:,:,slicePosition[2]]   # 100x100 array of function values
slicez = go.Surface(x=X, y=Y, z=Z, surfacecolor=surfz, cmin=vmin, cmax=vmax, showscale=False)

# --- create the YZ-slice
Y, Z = np.meshgrid(y,z)   # each is a 100x100 mesh covering [0,1] in each dimension
X = x[slicePosition[0]] * np.ones((100,100))   # 100x100 array of constant value
surfx = var[slicePosition[0],:,:]   # 100x100 array of function values
slicex = go.Surface(x=X, y=Y, z=Z, surfacecolor=surfx, cmin=vmin, cmax=vmax, showscale=True)

# --- create the XZ-slice
X, Z = np.meshgrid(x,z)
Y = y[slicePosition[1]] * np.ones((100,100))   # 100x100 array of constant value
surfy = var[:,slicePosition[1],:]
slicey = go.Surface(x=X, y=Y, z=Z, surfacecolor=surfy, cmin=vmin, cmax=vmax, showscale=False)

# --- plot the three slices
axis = dict(showbackground=True,  backgroundcolor="rgb(230, 230,230)",
            gridcolor="rgb(255, 255, 255)", zerolinecolor="rgb(255, 255, 255)")
layout = go.Layout(title='Orthogonal slices through volumetric data', 
                   width=900, height=900,
                   scene=go.Scene(xaxis=go.XAxis(axis),
                               yaxis=go.YAxis(axis), 
                               zaxis=go.ZAxis(axis), 
                               aspectratio=dict(x=1, y=1, z=1)))
fig = go.Figure(data=[slicez,slicey,slicex], layout=layout)
py.plot(fig,auto_open=False)
~~~

## Animation

For animations, you need to pass the third (`frames`) argument to `go.Figure()`:

~~~ {.python}
import plotly.plotly as py
import plotly.graph_objs as go
import numpy as np
t = np.linspace(0,10,100)
x, y = t/3*np.cos(t), t/3*np.sin(t)
line1 = go.Scatter(x=x, y=y, mode='lines', line=dict(width=10, color='blue'))
line2 = go.Scatter(x=x, y=y, mode='lines', line=dict(width=2, color='blue'))
layout = go.Layout(title='Moving dot',
                   xaxis=dict(range=[-10,10]),
                   yaxis=dict(scaleanchor="x", scaleratio=1),   # 1:1 aspect ratio
                   showlegend = False,
                   updatemenus= [{'type': 'buttons',
                                  'buttons': [{'label': 'Replay',
                                               'method': 'animate',
                                               'args': [None]}]}])
nframes = 20
s = np.linspace(0,10,nframes)
xdot, ydot = s/3*np.cos(s), s/3*np.sin(s)
frames = [dict(data=[go.Scatter(x=[xdot[k]], y=[ydot[k]],
                                mode='markers', marker=dict(color='red', size=30))])
          for k in range(nframes)]
# line1 will be shown before the first frame, line2 will be shown in each frame
fig = go.Figure(data=[line1,line2], layout=layout, frames=frames)
py.plot(fig,auto_open=False)
~~~

I could not find how to control the animation speed. Obviously, it should be via a keyword to either
`go.Figure()` or `py.plot`.

&nbsp;
