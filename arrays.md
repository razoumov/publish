<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Running Jupyter notebooks](#running-jupyter-notebooks)
- [Copying variables](#copying-variables)
- [Numpy](#numpy)
  - [Working with mathematical arrays in numpy](#working-with-mathematical-arrays-in-numpy)
  - [Indexing, slicing, and reshaping](#indexing-slicing-and-reshaping)
  - [Vectorized functions on array elements (aka universal functions = ufunc)](#vectorized-functions-on-array-elements-aka-universal-functions--ufunc)
  - [Aggregate functions](#aggregate-functions)
  - [Boolean indexing](#boolean-indexing)
  - [More numpy functionality](#more-numpy-functionality)
  - [External packages built on top of numpy](#external-packages-built-on-top-of-numpy)
- [Multi-dimensional labelled arrays and datasets with xarray](#multi-dimensional-labelled-arrays-and-datasets-with-xarray)
  - [Data array: simple example from scratch](#data-array-simple-example-from-scratch)
  - [Subsetting arrays](#subsetting-arrays)
  - [Plotting](#plotting)
  - [Vectorized operations](#vectorized-operations)
  - [Split your data into multiple independent groups](#split-your-data-into-multiple-independent-groups)
  - [Dataset: simple example from scratch](#dataset-simple-example-from-scratch)
  - [Time series data](#time-series-data)
  - [Adhering to climate and forecast (CF) NetCDF convention in spherical geometry](#adhering-to-climate-and-forecast-cf-netcdf-convention-in-spherical-geometry)
  - [Working with atmospheric data](#working-with-atmospheric-data)
  - [Plotting with cartopy](#plotting-with-cartopy)
- [Analysis and visualization of 3D multi-resolution data with yt](#analysis-and-visualization-of-3d-multi-resolution-data-with-yt)
  - [Small time-dependent dataset](#small-time-dependent-dataset)
  - [Generic array data](#generic-array-data)
  - [Bigger sample multi-resolution datasets](#bigger-sample-multi-resolution-datasets)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Running Jupyter notebooks

You can find these notes at http://bit.ly/wgarrays.

Python pros                                 | Python cons
--------------------------------------------|------------------------
elegant scripting language                  | slow (interpreted, dynamically typed)
powerful, compact constructs for many tasks |
very popular across all fields              |
huge number of external libraries           |

Python programs are plain text files, stored with the .py extension.

Many ways to run Python commands:

* from the inside a Unix shell can start a Python shell and type commands
* running Python scripts saved in *.py files
* from Jupyter (formerly iPython) notebooks - stored as JSON files, displayed as HTML

Today we will use a Jupyter notebook. You have several options:

1. if you have a university computer ID, go to https://syzygy.ca and under Launch select your institution, then fill in your credentials
1. if you have a Google account, go to https://syzygy.ca and under Launch select either Cybera or PIMS, then log in
1. if you have a GitHub account, go to https://westgrid.syzygy.ca
1. if you have Python+Jupyter installed locally on your machine, then you can start it locally from your shell by typing
  `jupyter notebook`; you will need the following Python packages installed on your computer: numpy, pandas,
  scikit-image, matplotlib, xarray, nc-time-axis, cartopy, netcdf (for reading/writing NetCDF files), yt

This will open a browser page pointing to the Jupyter server (remote except for the last option). Click on New ->
Python 3. Explain: tab completion, annotating code, displaying figures inside the notebook.

* Esc - leave the cell (border becomes blue) to the control mode
* "A" - insert a cell above the current cell
* "B" - insert a cell below the current cell
* "X" - delete the current cell
* "M" - turn the current cell into the markdown cell
* "H" - to display help
* Enter - re-enter the cell (border becomes green) from the control mode
* can enter Latex equations in a markdown cell, e.g. $int_0^\infty f(x)dx$

~~~ {.python}
print(1/2)   # to run all commands in the cell, either use the Run button, or press shift+return
~~~

# Copying variables

With simple/primitive variables in Python, assigning `var2 = var1` will create a new object in memory `var2`.

> Note: With more complex objects such as lists or numpy arrays, its name would be a pointer. E.g. with lists,
> `initial` and `new` below really point to the same list in memory:
> ~~~
> initial = [1,2,3]
> new = initial        # create a pointer to the same object
> initial.append(4)    # change the original list to [1, 2, 3, 4]
> print(new)           # [1, 2, 3, 4]
> new = initial[:]     # one way to create a new object in memory
> import copy
> new = copy.deepcopy(initial)   # another way to create a new object in memory
> ~~~









# Numpy

As you know, Python is not statically typed, i.e. variables can change their type on the fly:

~~~
a = 5
a = 'apple'
print(a)
~~~

This makes Python very flexible. Out of these variables you form 1D lists, and these can be inhomogeneous:

~~~
a = [1, 2, 'Vancouver', ['Earth', 'Moon'], {'list': 'an ordered collection of values'}]
a[1] = 'Sun'
a
~~~

Python lists are very general and flexible, which is great for high-level programming, but it comes at a cost. The
Python interpreter can't make any assumptions about what will come next in a list, so it treats everything as a generic
object with its own type and size. As lists get longer, eventually performance takes a hit.

Python does not have any mechanism for a uniform/homogeneous list, where -- to jump to element #1000 -- you just take
the memory address of the very first element and then increment it by (element size in bytes) x 999. **Numpy** library
fills this gap by adding the concept of homogenous collections to python -- `numpy.ndarray`'s -- which are
multi-dimensional, homogeneous arrays of fixed-size items (most commonly numbers).

1. This brings large performance benefits!
  - no reading of extra bits (type, size, reference count)
  - no type checking
  - contiguous allocation in memory
1. numpy lets you work with mathematical arrays.

Lists and numpy arrays behave very differently:

~~~
a = [1, 2, 3, 4]
b = [5, 6, 7, 8]
a + b              # this will concatenate two lists: [1,2,3,4,5,6,7,8]
~~~

~~~
import numpy as np
na = np.array([1, 2, 3, 4])
nb = np.array([5, 6, 7, 8])
na + nb            # this will sum two vectors element-wise: array([6,8,10,12])
na * nb            # element-wise product
~~~

## Working with mathematical arrays in numpy

Numpy arrays have the following attributes:

- ndim = the number of dimensions
- shape = a tuple giving the sizes of the dimensions
- size = the total number of elements
- dtype = the data type
- itemsize = the size (bytes) of individual elements
- nbytes = the total memory (bytes) occupied by the ndarray
- strides = tuple of bytes to step in each dimension when traversing an array
- data = memory address of the array

~~~
a = np.arange(10)      # 10 integer elements 0..9
a.ndim      # 1
a.shape     # (10,)
a.nbytes    # 80
a.dtype     # dtype('int64')
b = np.arange(10, dtype=np.float)
b.dtype     # dtype('float64')
~~~

In numpy there are many ways to create arrays:

~~~
np.arange(11,20)               # 9 integer elements 11..19
np.linspace(0, 1, 100)         # 100 numbers uniformly spaced between 0 and 1 (inclusive)
np.linspace(0, 1, 100).shape
np.zeros(100, dtype=np.int)    # 1D array of 100 integer zeros
np.zeros((5, 5), dtype=np.float64)     # 2D 5x5 array of floating zeros
np.ones((3,3,4), dtype=np.float64)     # 3D 3x3x4 array of floating ones
np.eye(5)            # 2D 5x5 identity/unit matrix (with ones along the main diagonal)
~~~

You can create random arrays:

~~~
np.random.randint(0, 10, size=(4,5))    # 4x5 array of random integers in the half-open interval [0,10)
np.random.random(size=(4,3))            # 4x3 array of random floats in the half-open interval [0.,1.)
np.random.rand(3, 3)       # 3x3 array drawn from a uniform [0,1) distribution
np.random.randn(3, 3)      # 3x3 array drawn from a normal (Gaussian with x0=0, sigma=1) distribution
~~~

## Indexing, slicing, and reshaping

For 1D arrays:

~~~
a = np.linspace(0,1,100)
a[0]        # first element
a[-2]       # 2nd to last element
a[5:12]     # values [5..12), also a numpy array
a[5:12:3]   # every 3rd element in [5..12), i.e. elements 5,8,11
a[::-1]     # array reversed
~~~

Similar for multi-dimensional arrays:

~~~
b = np.reshape(np.arange(100),(10,10))      # form a 10x10 array from 1D array
b[0:2,1]      # first two rows, second column
b[:,-1]       # last column
b[5:7,5:7]    # 2x2 block
~~~

~~~
a = np.array([1, 2, 3, 4])
b = np.array([4, 3, 2, 1])
np.vstack((a,b))   # stack them vertically into a 2x4 array (use a,b as rows)
np.hstack((a,b))   # stack them horizontally into a 1x8 array
np.column_stack((a,b))    # use a,b as columns
~~~

## Vectorized functions on array elements (aka universal functions = ufunc)

One of the big reasons for using numpy is so you can do fast numerical operations on a large number of elements. The
result is another `ndarray`. In many calculations you can use replace the usual `for`/`while` loops with functions on
numpy elements.

~~~
a = np.arange(100)
a**2          # each element is a square of the corresponding element of a
np.log10(a+1)     # apply this operation to each element
(a**2+a)/(a+1)    # the result should effectively be a floating-version copy of a
np.arange(10) / np.arange(1,11)  # this is np.array([ 0/1, 1/2, 2/3, 3/4, ..., 9/10 ])
~~~

> **[Exercise](./solai.md)**: Let's verify the equation
> <img src="https://raw.githubusercontent.com/razoumov/publish/master/eq001.png" height="80" />
> using summation of elements of an `ndarray`.
>
> **Hint**: Start with the first 10 terms `k = np.arange(1,11)`. Then try the first 30 terms.







An extremely useful feature of ufuncs is the ability to operate between arrays of different sizes and shapes, a set of
operations known as *broadcasting*.

~~~
a = np.array([0, 1, 2])    # 1D array
b = np.ones((3,3))         # 2D array
a + b          # `a` is stretched/broadcast across the 2nd dimension before addition;
               # effectively we add `a` to each row of `b`
~~~

In the following example both arrays are broadcast from 1D to 2D to match the shape of the other:

~~~
a = np.arange(3)                     # 1D row;                a.shape is (3,)
b = np.arange(3).reshape((3,1))      # effectively 1D column; b.shape is (3, 1)
a + b                                # the result is a 2D array!
~~~

Numpy's broadcast rules are:

1. the shape of an array with fewer dimensions is padded with 1's on the left
1. any array with shape equal to 1 in that dimension is stretched to match the other array's shape
1. if in any dimension the sizes disagree and neither is equal to 1, an error is raised

~~~
Example 1:
==========
a: (2,3)  ->  (2,3)  ->  (2,3)
b: (3,)   ->  (1,3)  ->  (2,3)
                                ->  (2,3)

Example 2:
==========
a: (3,1)  ->  (3,1)  ->  (3,3)
b: (3,)   ->  (1,3)  ->  (3,3)
                                ->  (3,3)

Example 3:
==========
a: (3,2)  ->  (3,2)  ->  (3,2)
b: (3,)   ->  (1,3)  ->  (3,3)
                                ->  error
"ValueError: operands could not be broadcast together with shapes (3,2) (3,)"
~~~

> Note on numpy speed: As chance would have it, last week I was working with a spherical dataset describing Earth's
> mantle convection. It is on a spherical grid with 13e6 grid points. For each grid point, I was converting from the
> spherical (lateral - radial - longitudinal) velocity components to the Cartesian velocity components. For each point
> this is a matrix-vector multiplication. Doing this by hand with Python's `for` loops would take many hours for 13e6
> points. I used numpy to vectorize in one dimension, and that cut the time to ~5 mins. At first glance, a more complex
> vectorization would not work, as numpy would have to figure out which dimension goes where. Writing it carefully and
> following the broadcast rules I made it work, with the correct solution at the end -- while the total compute time
> went down to a few seconds!

Let's use broadcasting to plot a 2D function with matplotlib:

~~~
%matplotlib inline
import matplotlib.pyplot as plt
plt.figure(figsize=(12,12))
x = np.linspace(0, 5, 50)
y = np.linspace(0, 5, 50).reshape(50,1)
z = np.sin(x)**8 + np.cos(5+x*y)*np.cos(x)    # broadcast in action!
plt.imshow(z)
plt.colorbar(shrink=0.8)
~~~

**[Exercise](sol03.md):** Use numpy broadcasting to build a 3D array from three 1D ones.








## Aggregate functions

Aggregate functions take an ndarray and reduce it along one (or more) axes. E.g., in 1D:

~~~
a = np.linspace(1, 2, 100)
a.mean()     # arithmetic mean
a.max()      # maximum value
a.argmax()   # index of the maximum value
a.sum()      # sum of all values
a.prod()     # product of all values
~~~

Or in 2D:

~~~
b = np.arange(25).reshape(5,5)
>>> b.sum()
300
b.sum(axis=0)   # add rows
b.sum(axis=1)   # add columns
~~~

## Boolean indexing

~~~
a = np.linspace(1, 2, 100)
a < 1.5
a[a < 1.5]    # will only return those elements that meet the condition
a[a < 1.5].shape
a.shape
~~~

## More numpy functionality

Numpy provides many standard linear algebra algorithms: matrix/vector products, decompositions, eigenvalues, solving
linear equations, e.g.

~~~
a = np.array([[3,1], [1,2]])
b = np.array([9,8])
x = np.linalg.solve(a, b)
x
np.allclose(np.dot(a, x),b)    # check the solution
~~~

## External packages built on top of numpy

A lot of other packages are built on top of numpy. Many image-processing libraries use numpy data structures underneath,
e.g. **scikit-image**:

~~~
import skimage.io        # scikit-image is a collection of algorithms for image processing
image = skimage.io.imread(fname="https://raw.githubusercontent.com/razoumov/publish/master/grids.png")
image.shape       # it's a 1024^2 image, with (R,G,B,\alpha) channels
~~~

Let's plot this image using matplotlib:

~~~
%matplotlib inline
import matplotlib.pyplot as plt
plt.figure(figsize=(10,10))
plt.imshow(image[:,:,2], interpolation='nearest')
plt.colorbar(orientation='vertical', shrink=0.75, aspect=50)
~~~

Using numpy, you can easily manipulate pixels:

~~~
image[:,:,2] = 255 - image[:,:,2]
~~~

and then rerun the previous (matplotlib) cell.

Another example of a package built on top of numpy is **pandas**, for working with 2D tables. Going further, **xarray**
package for working with labelled multi-dimensional arrays was built on top of both numpy and pandas. And **yt** package
for analysis and visualization of 3D multi-resolution volumetric data is also based on numpy.










# Multi-dimensional labelled arrays and datasets with xarray

Xarray library is built on top of numpy and pandas, and it brings the power of pandas to multi-dimensional arrays. There
are two main data structures in xarray:

- xarray.DataArray is a fancy, labelled version of numpy.ndarray
- xarray.Dataset is a collection of multiple xarray.DataArray's that share dimensions

## Data array: simple example from scratch

```python
import xarray as xr
import numpy as np
import pandas as pd
data = xr.DataArray(
    np.random.random(size=(4,3)),
    dims=("y","x"),  # dimension names (row,col); we want `y` to represent rows and `x` columns
    coords={"x": [10,11,12], "y": [10,20,30,40]}  # coordinate labels/values
)
data
```

We can access various attributes of this array:

```python
data.values                 # the 2D numpy array
data.values[0,0] = 0.53     # can modify in-place
data.dims                   # ('y', 'x')
data.coords               # all coordinates, cannot modify!
data.coords['x'][1]       # a number
data.x[1]                 # the same
```

Let's add some arbitrary metadata:

```python
data.attrs = {"author": "AR", "date": "2020-08-26"}
data.attrs["name"] = "density"
data.attrs["units"] = "g/cm^3"
data.x.attrs["units"] = "cm"
data.y.attrs["units"] = "cm"
data.attrs    # all attributes
data          # all attributes show here as well
data.x        # only `x` attributes
```

## Subsetting arrays

We can subset using the usual Python square brackets:

```python
data[0,:]     # first row
data[:,-1]    # last column
```

In addition, xarray provides these functions:

- isel() selects by index, could be replaced by [index1] or [index1,...]
- sel() selects by value
- interp() interpolates by value

```python
data.isel()      # same as `data`
data.isel(y=1)   # second row
data.isel(y=0, x=[-2,-1])    # first row, last two columns
```

```python
data.x.dtype     # it is integer
data.sel(x=10)   # certain value of `x`
data.y   # array([10, 20, 30, 40])
data.sel(y=slice(15,30))   # only values with 15<=y<=30 (two rows)
```

There are aggregate functions, e.g.

```python
meanOfEachColumn = data.mean(dim='y')    # apply mean over y
spatialMean = data.mean()
spatialMean = data.mean(dim=['x','y'])   # same
```

Finally, we can interpolate:

```python
data.interp(x=10.5, y=10)    # first row, between 1st and 2nd columns
data.interp(x=10.5, y=15)    # between 1st and 2nd rows, between 1st and 2nd columns
?data.interp                 # can use different interpolation methods
```

## Plotting

Matplotlib is integrated directly into xarray:

```python
data.plot(size=8)                         # 2D heatmap
data.isel(x=0).plot(marker="o", size=8)   # 1D line plot
```

## Vectorized operations

You can perform element-wise operations on xarray.DataArray like with numpy.ndarray:

```python
data + 100      # element-wise like numpy arrays
(data - data.mean()) / data.std()    # normalize the data
data - data[0,:]      # use numpy broadcasting => subtract first row from all rows
```

## Split your data into multiple independent groups

```python
data.groupby("x")   # 3 groups with labels 10, 11, 12
data.groupby("x").map(lambda v: v-v.min())   # apply separately to each group
            # from each column (fixed x) subtract the smallest value in that column
```

## Dataset: simple example from scratch

Let's initialize two 2D arrays with the identical dimensions:

```python
coords = {"x": np.linspace(0,1,5), "y": np.linspace(0,1,5)}
temp = xr.DataArray(      # first 2D array
    20 + np.random.randn(5,5),
    dims=("y","x"),
    coords=coords
)
pres = xr.DataArray(       # second 2D array
    100 + 10*np.random.randn(5,5),
    dims=("y","x"),
    coords=coords
)
```

From these we can form a dataset:

```python
ds = xr.Dataset({"temperature": temp, "pressure": pres,
                 "bar": ("x", 200+np.arange(5)), "pi": np.pi})
ds
```

As you can see, `ds` includes two 2D arrays on the same grid, one 1D array on `x`, and one number:

```python
ds.temperature   # 2D array
ds.bar           # 1D array
ds.pi            # one element
```

Subsetting works the usual way:

```python
ds.sel(x=0)     # each 2D array becomes 1D array, the 1D array becomes a number, plus a number
ds.temperature.sel(x=0)     # 'temperature' is now a 1D array
ds.temperature.sel(x=0.25, y=0.5)     # one element of `temperature`
```

We can save this dataset to a file:

```python
%pip install netcdf4
ds.to_netcdf("test.nc")
new = xr.open_dataset("test.nc")   # try reading it
```

We can even try opening this 2D dataset in ParaView - select (x,y) and deselect Spherical.

> **[Exercise](sol04.md):** Recall the 2D function we plotted when we were talking about numpy's array
> broadcasting. Let's scale it to a unit square x,y∈[0,1]:
> ~~~
> x = np.linspace(0, 1, 50)
> y = np.linspace(0, 1, 50).reshape(50,1)
> z = np.sin(5*x)**8 + np.cos(5+25*x*y)*np.cos(5*x)
> ~~~
> This is will our image at z=0. Then rotate this image 90 degrees (e.g. flip x and y), and this will be our function at
> z=1. Now interpolate linearly between z=0 and z=1 to build a 3D function in the unit cube x,y,z∈[0,1]. Check what the
> function looks like at intermediate z. Write out a NetCDF file with the 3D function.









## Time series data

In xarray you can work with time-dependent data. Xarray accepts pandas time formatting,
e.g. `pd.to_datetime("2020-09-10")` would produce a timestamp. To produce a time range, we can use:

~~~
import pandas as pd
time = pd.date_range("2000-01-01", freq="D", periods=365*3+1)    # 2000-Jan to 2002-Dec (3 full years)
time
time.month    # same length (1096), but each element is replaced by the month number
time.day      # same length (1096), but each element is replaced by the day-of-the-month
?pd.date_range
~~~

Using this `time` construct, let's initialize a time-dependent dataset that contains a scalar temperature variable (no
space) mimicking seasonal change. We can do this directly without initializing an xarray.DataArray first -- we just need
to specify what this temperature variable depends on:

~~~
import xarray as xr
import numpy as np
ntime = len(time)
temp = 10 + 5*np.sin((250+np.arange(ntime))/365.25*2*np.pi) + 2*np.random.randn(ntime)
ds = xr.Dataset({ "temperature": ("time", temp),        # it's 1D function of time
                  "time": time })
ds.temperature.plot(size=8)
~~~

We can do the usual subsetting:

~~~
ds.isel(time=100)   # 101st timestep
ds.sel(time="2002-12-22")
~~~

Time dependency in xarray allows resampling with a different timestep:

~~~
ds.resample(time='7D')    # 1096 times -> 157 time groups
weekly = ds.resample(time='7D').mean()     # compute mean for each group
weekly.temperature.plot(size=8)
~~~

Now, let's combine spatial and time dependency and construct a dataset containing two 2D variables (temperature and
pressure) varying in time. The time dependency is baked into the coordinates of these xarray.DataArray's and should come
before the spatial coordinates:

~~~
time = pd.date_range("2020-01-01", freq="D", periods=91) # January - March 2020
ntime = len(time)
n = 100      # spatial resolution in each dimension
axis = np.linspace(0,1,n)
X, Y = np.meshgrid(axis,axis)   # 2D Cartesian meshes of x,y coordinates
initialState = (1-Y)*np.sin(np.pi*X) + Y*(np.sin(2*np.pi*X))**2
finalState =   (1-X)*np.sin(np.pi*Y) + X*(np.sin(2*np.pi*Y))**2
f = np.zeros((ntime,n,n))
for t in range(ntime):
    z = (t+0.5) / ntime   # dimensionless time from 0 to 1
    f[t,:,:] = (1-z)*initialState + z*finalState

coords = {"time": time, "x": axis, "y": axis}
temp = xr.DataArray(
    20 + f,       # this 2D array varies in time from initialState to finalState
    dims=("time","y","x"),
    coords=coords
)
pres = xr.DataArray(   # random 2D array
    100 + 10*np.random.randn(ntime,n,n),
    dims=("time","y","x"),
    coords=coords
)
ds = xr.Dataset({"temperature": temp, "pressure": pres})
ds.sel(time="2020-03-15").temperature.plot(size=8)   # temperature distribution on a specific date
ds.to_netcdf("evolution.nc")
~~~

The file `evolution.nc` should be 100^2 x 2 variables x 8 bytes x 91 steps = 14MB. We can load it into ParaView and play
back the pressure and temperature!






## Adhering to climate and forecast (CF) NetCDF convention in spherical geometry

So far we've been working with datasets in Cartesian coordinates. How about spherical geometry -- how do we initialize
and store a dataset in spherical coordinates (longitude - latitude - elevation)? Very easy: define these coordinates and
your data arrays on top, put everything into an xarray dataset, and then specify the following units:

~~~
ds.lat.attrs["units"] = "degrees_north"   # this line is important to adhere to CF convention
ds.lon.attrs["units"] = "degrees_east"    # this line is important to adhere to CF convention
~~~

**[Exercise](sol05.md):** Let's do it! Create a small (one-degree horizontal + some vertical resolution), stationary (no
time dependency) dataset in spherical geometry with one 3D variable and write it to `spherical.nc`. Load it into
ParaView to make sure the geometry is spherical.












## Working with atmospheric data

I took one of the ECCC historical model datasets (contains only the near-surface air temperature) published on the CMIP6
Data-Archive and reduced its size picking only a subset of timesteps:

~~~
import xarray as xr
data = xr.open_dataset('/Users/razoumov/tmp/xarray/atmosphere/tas_Amon_CanESM5_historical_r1i1p2f1_gn_185001-201412.nc')
data.sel(time=slice('2001', '2020')).to_netcdf("tasReduced.nc")   # last 168 steps
~~~

Let's download this file in the terminal:

~~~
wget http://bit.ly/atmosdata -O tasReduced.nc    # 4.3MB
~~~

First, quickly check this dataset in ParaView (use Dimensions = (lat,lon)).

~~~
data = xr.open_dataset('tasReduced.nc')
data   # this is a time-dependent 2D dataset: print out the metadata, coordinates, data variables
data.time   # time goes monthly from 2001-01-16 to 2014-12-16
data.tas    # metadata for the data variable (time: 168, lat: 64, lon: 128)
data.tas.shape      # (168, 64, 128) = (time, lat, lon)
data.height         # at the fixed height=2m
~~~

These five lines all produce the same result:

~~~
data.tas[0] - 273.15   # take all values in the second and third dims, convert to Celsius
data.tas[0,:] - 273.15
data.tas[0,:,:] - 273.15
data.tas.isel(time=0) - 273.15
air = data.tas.sel(time='2001-01-16') - 273.15
~~~

These two lines produce the same result (1D vector of temperatures as a function of longitude):

~~~
data.tas[0,5]
data.tas.isel(time=0, lat=5)
~~~

Check temperature variation in the last step:

~~~
air = data.tas.isel(time=-1) - 273.15   # last timestep, to celsius
air.shape    # (64, 128)
air.min(), air.max()   # -43.550903, 36.82956
~~~

Selecting data is slightly more difficult with approximate floating coordinates:

~~~
data.tas.lat
data.tas.lat.dtype
data.tas.isel(lat=0)    # the first value lat=-87.86
data.lat[0]   # print the first latitude and try to use it below
data.tas.sel(lat=-87.86379884)    # does not work due to floating precision
data.tas.sel(lat=data.lat[0])     # this works
latSlice = data.tas.sel(lat=slice(-90,-80))    # only select data in a slice lat=[-90,-80]
latSlice.shape    # (168, 3, 128) - 3 latitudes in this slice
~~~

Multiple ways to select time:

~~~
data.time[-10:]   # last ten times
air = data.tas.sel(time='2014-12-16') - 273.15    # last date
air = data.tas.sel(time='2014') - 273.15    # select everything in 2014
air.shape     # 12 steps
air.time
air = data.tas.sel(time='2014-01') - 273.15    # select everything in January 2014
~~~

Aggregate functions:

~~~
meanOverTime = data.tas.mean(dim='time') - 273.15
meanOverSpace = data.tas.mean(dim=['lat','lon']) - 273.15     # mean over space for each timestep
meanOverSpace.shape     # time series (168,)
meanOverSpace.plot(marker="o", size=8)     # calls matplotlib.pyplot.plot
~~~

Interpolate to a specific location:

~~~
victoria = data.tas.interp(lat=48.43, lon=360-123.37)
victoria.shape   # (168,) only time
victoria.plot(marker="o", size=8)      # simple 1D plot
victoria.sel(time=slice('2001','2020')).plot(marker="o", size=8)   # zoom in on the 21st-century points, see seasonal variations
~~~

Let's plot in 2D:

~~~
air = data.tas.isel(time=-1) - 273.15   # last timestep
air.time
air.plot(size=8)     # 2D plot, very poor resolution (lat: 64, lon: 128)
air.plot(size=8, y="lon", x="lat")     # can specify which axis is which
~~~

What if we have time-dependency in the plot?

~~~
a = data.tas[-6:] - 273.15      # last 6 timesteps => 3D dataset => which coords to use for what?
a.plot(x="lon", y="lat", col="time", col_wrap=3)
~~~

Breaking into groups and applying a function to each group:

~~~
len(data.time)     # 168 steps
data.tas.groupby("time")   # 168 groups
def standardize(x):
    return (x - x.mean()) / x.std()
standard = data.tas.groupby("time").map(standardize)   # apply this function to each group
standard.shape    # (1980, 64, 128) same shape as the original but now normalized over each group
~~~

## Plotting with cartopy

Cartopy lets you process your geospatial data in order to produce maps, e.g. using various map projections. All plotting
is still being done by Matplotlib.

~~~
import cartopy.crs as ccrs
import matplotlib.pyplot as plt
import cartopy.feature as cfeature
fig = plt.figure(figsize=(12,12))
ax = fig.add_subplot(111, projection=ccrs.PlateCarree())
ax.coastlines()
ax.coastlines(resolution='50m', color='gray', linewidth=2)   # valid scales are "110m", "50m", "10m"
ax.add_feature(cfeature.OCEAN)
ax.add_feature(cfeature.LAND)
ax.add_feature(cfeature.BORDERS, linestyle=':')
~~~

Try the following projections (and bring online help on these):

- PlateCarree(0) is the equi-rectangular/Cartesian projection (meridians are vertical straight lines)
- Orthographic(-100,55)
- Mollweide(0)
- Robinson(0)
- InterruptedGoodeHomolosine(0)

[More information](https://scitools.org.uk/cartopy/docs/latest/crs/projections.html#cartopy-projections) on cartopy projections.

Let's only plot North America:

~~~
fig = plt.figure(figsize=(12,12))
ax = fig.add_subplot(111, projection=ccrs.Mollweide(-100))
ax.coastlines()
ax.add_feature(cfeature.OCEAN)
ax.add_feature(cfeature.LAND)
ax.add_feature(cfeature.BORDERS, linestyle=':')
ax.set_extent([-160, -51, 5, 85])
~~~

Most matplotlib plotting functions will work with cartopy projections: lines (`plot`), heatmaps and images (`plot` or
`imshow`), scatter plots (`scatter`), vector plots (`quiver`). Let's plot our 2D atmospheric data with Cartopy
projections. Read the data again:

~~~
import xarray as xr
data = xr.open_dataset('tasReduced.nc')
temp = data.tas.sel(time="2014-12-16") - 273.15   # extract last timestep, convert to Celsius
temp.shape      # (1, 64, 128)
temp.coords    # the coordinates are 1D, so plotting is easy
~~~

The data is defined on a 2D Cartesian projection. First, let's plot data in the same projection:

~~~
import cartopy.crs as ccrs
import matplotlib.pyplot as plt
fig = plt.figure(figsize=(14,12))
ax = fig.add_subplot(111, projection=ccrs.PlateCarree())
ax.coastlines()
ax.gridlines()
temp.plot(ax=ax, cbar_kwargs={'shrink': 0.4})   # since we are using PlateCarree(), Cartopy assumes
                                                # that the data is also in PlateCarree() coordinates, which is true
~~~

Now let's switch the projection. Then we have to tell Cartopy our data's coordinate system explicitly with `transform`:

~~~
fig = plt.figure(figsize=(14,12))
ax = fig.add_subplot(111, projection=ccrs.InterruptedGoodeHomolosine())   # use this projection
ax.coastlines()
ax.gridlines()
temp.plot(ax=ax, transform=ccrs.PlateCarree(), cbar_kwargs={'shrink': 0.4})
            # `transform` tells Cartopy the coordinate system of our data
~~~

Let's switch the projection and zoom in on Vancouver Island:

~~~
ax = fig.add_subplot(111, projection=ccrs.Mollweide(-125))
ax.coastlines()
ax.gridlines()
temp.plot(ax=ax, transform=ccrs.PlateCarree(), cbar_kwargs={'shrink': 0.4})
ax.set_extent([-129, -122, 46, 53])
~~~

We can see that our data is really low resolution.








# Analysis and visualization of 3D multi-resolution data with yt

[YT](https://yt-project.org) is a Python package for analyzing and visualizing volumetric, multi-resolution data. It
supports a number of discretizations: structured, unstructured, variable-resolution (curvilinear), particle data. It was
initially written for analysing Enzo (galaxy formation models) output data but was later adapted to understand other
data formats from astrophysics and beyond (astrophysics, seismology, nuclear engineering, molecular dynamics,
oceanography).

To install yt, use one of these lines in the terminal:

~~~
conda install -c conda-forge yt
pip install yt
git clone https://github.com/yt-project/yt && cd yt && python setup.py develop
~~~

on in Python in the Jupyter notebook:

~~~
%pip install yt
~~~

## Small time-dependent dataset

Get some sample data:

~~~
wget http://yt-project.org/data/EnzoKelvinHelmholtz.tar.gz   # 4.3MB, 2D Kelvin-Helmholtz test, 11 timesteps
tar xvfz EnzoKelvinHelmholtz.tar.gz && rm EnzoKelvinHelmholtz.tar.gz
~~~

~~~
import yt
ds = yt.load('EnzoKelvinHelmholtz/DD0010/DD0010')
ds.print_stats()
ds.field_list
slc = yt.SlicePlot(ds, normal='z', fields='Density').annotate_grids()
slc.show()
~~~

~~~
ts = yt.load('EnzoKelvinHelmholtz/DD????/DD????')
for ds in ts:
    name = str(ds)[-4:]
    slc = yt.SlicePlot(ds, normal='z', fields='Density').annotate_grids()
    slc.save('frame'+name+'.png')
~~~

Create the movie in the terminal:

~~~
ffmpeg -r 30 -i frame%04d.png -c:v libx264 -pix_fmt yuv420p -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" evolve.mp4
~~~

## Generic array data

This is Python, so we can start with any array in any form. Let's load some NetCDF data.

~~~
wget https://github.com/razoumov/publish/raw/master/sineEnvelope.nc   # 3.8MB
~~~

~~~
%pip install netCDF4
import yt, numpy as np
from netCDF4 import Dataset
all = Dataset('sineEnvelope.nc', 'r')
rho = all.variables['density'][:,:,:]
data = {'density': rho}
bbox = np.array([[0,1],[0,1],[0,1]])
ds = yt.load_uniform_grid(data=data, domain_dimensions=rho.shape, length_unit=1., bbox=bbox)
ds.print_stats()
slc = yt.SlicePlot(ds, 'x', 'density', center='c')
slc.set_log('density', False) # linear colourmap
slc.show()
~~~

~~~
sc = yt.create_scene(ds, 'density')
source = sc[0]              # the object in the scene to be rendered
source.set_field('density') # set the field
source.set_log(False)       # use linear colourmap
~~~

Build the transfer function with a cyan spike:

~~~
bounds = (0,1)
tf = yt.ColorTransferFunction(x_bounds=bounds)
tf.add_gaussian(location=0.15, width=0.005, height=[0.5,1,1,1]) # [r,g,b,alpha]
print(tf)
tf.show()
~~~

Apply our transfer function to the source:

~~~
source.tfh.tf = tf           # tfh is TransferFunctionHelper
source.tfh.bounds = bounds
source.tfh.plot('transferFunction.png', profile_field='density')
sc.camera.resolution = (1024,1024)
sc.camera.position=[1,1,1.5]
sc.show(sigma_clip=3)   # adjust brightness
~~~

## Bigger sample multi-resolution datasets

More detailed slides at https://westgrid.github.io/trainingMaterials/materials/yt20181121.pdf -- you will need to run
these on your own computer due to the disk quota in syzygy.ca.
