You can find a copy of these notes at http://bit.ly/chAnimation.

In this session we will be talking about camera animation for a 3D static scene, both in ParaView and
VisIt. Camera animation is an effective method to produce an engaging animation even when you do not have
any time evolution.

In both ParaView and VisIt you can produce animations from the GUI, however, scripting will give you much
more control over what you can do inside your animation.

# Camera animation in ParaView

Let's open our visualization from the ParaView state file:

~~~{.bash}
cd ~/visualizeThis
paraview --state=bladesWithLines.pvsm
~~~

Now let's take a look at some commands inside ParaView's Python shell:

~~~{.python}
help(GetActiveCamera)
dir(GetActiveCamera())
help(GetActiveCamera().Azimuth)
~~~

Normally, this is how you would rotate your visualization around the vertical axis:

~~~{.python}
camera = GetActiveCamera()
camera.Azimuth(10)
Render() or SaveScreenshot('filename.png')
~~~

However, we should not rotate now, since CenterOfRotation is off-focus:

~~~{.python}
v1 = GetActiveView()
print v1.CameraFocalPoint
print v1.CameraParallelScale
print v1.CameraPosition
print v1.CameraViewAngle
print v1.CameraViewUp
print v1.CenterOfRotation
~~~

See how in the output CameraFocalPoint and CenterOfRotation are different:

~~~
[0.228557134725041, 2.47726112917046, 0.658093579072617]
4.25832870332
[-14.0557805400554, 9.23241645115292, 5.24329881316648]
30.0
[0.25357815545749796, -0.11566287189265197, 0.9603750408774258]
[1.8223249912262, -0.010200023651123, 0.0]
~~~~

Let's reset the camera to include the entire scene but preserve orientation. This will rescale the
picture and put CenterOfRotation into CameraFocalPoint:
	
~~~{.python}
ResetCamera()
~~~

~~~{.python}
print v1.CameraFocalPoint
print v1.CameraParallelScale
print v1.CameraPosition
print v1.CameraViewAngle
print v1.CameraViewUp
print v1.CenterOfRotation
~~~

Notice that CameraFocalPoint and CenterOfRotation are now the same:

~~~
[1.7140189409255981, 10.003308773040771, 0.7980973720550537]
25.6202838711
[-84.22786381056636, 50.64577885934507, 28.38503766931171]
30.0
[0.25357815545749796, -0.11566287189265197, 0.9603750408774258]
[1.7140189409255981, 10.003308773040771, 0.7980973720550537]
~~~

Now we could do rotation, but probably we don't want to as the picture is somewhat small. Let's move the
camera a little bit closer to (CenterOfRotation = CameraFocalPoint):

~~~{.python}
v1.CameraPosition = [(0.6*a + 0.4*b) for a, b in zip(v1.CameraPosition,v1.CameraFocalPoint)]
Render()
~~~

Now let's rotate our visualization by 10 degrees:

~~~{.python}
camera = GetActiveCamera()
camera.Azimuth(10)
Render()
~~~

The image was rendered but not saved to disk. Let's rotate around the vertical axis by 360 degrees in 50
steps saving each frame as a PNG image:

~~~{.python}
nframes = 50
for i in range(nframes):
    print v1.CameraPosition
    camera.Azimuth(360./nframes)   # rotate by 7.2 degrees
    SaveScreenshot('/Users/razoumov/visualizeThis/frame%04d'%(i)+'.png')
~~~

We can merge the frames into a Quicktime-compatible MP4 movie at 10 fps:

~~~{.bash}
ffmpeg -r 10 -i frame%04d.png -c:v libx264 -pix_fmt yuv420p -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" spin.mp4
open spin.mp4
~~~

Let's reset the camera and try to fly 1/3 of the way towards the focal point:

~~~{.python}
ResetCamera()
initialCameraPosition = v1.CameraPosition[:]   # force a real copy, not a pointer
nframes = 50
for i in range(nframes):
    coef = float(i+0.5)/float(3*nframes)  # runs from 0 to 1/3
    print coef, v1.CameraPosition
    v1.CameraPosition = [((1.-coef)*a + coef*b) for a, b in zip(initialCameraPosition,v1.CameraFocalPoint)]
    SaveScreenshot('/Users/razoumov/visualizeThis/frame%04d'%(i)+'.png')
~~~

Again, we can merge the frames into a Quicktime-compatible MP4 movie:

~~~{.bash}
ffmpeg -r 10 -i frame%04d.png -c:v libx264 -pix_fmt yuv420p -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" approach.mp4
open approach.mp4
~~~

For the final ParaView animation, let's reset the camera and move the focal point and the center of
rotation 80% of the way towards the turbine blades:

~~~{.python}
ResetCamera()
v1.CameraFocalPoint[1] -= 10   # as the blades are centered towards positive y
v1.CenterOfRotation = v1.CameraFocalPoint[:]
v1.CameraPosition = [(0.2*a + 0.8*b) for a, b in zip(v1.CameraPosition,v1.CameraFocalPoint)]
Render()
~~~

Now let's spin around the blades:

~~~{.python}
camera = GetActiveCamera()
nframes = 50
for i in range(nframes):
    camera.Azimuth(360./nframes)
    SaveScreenshot('/Users/razoumov/visualizeThis/frame%04d'%(i)+'.png')
~~~

Let's make the final movie:

~~~{.bash}
ffmpeg -r 10 -i frame%04d.png -c:v libx264 -pix_fmt yuv420p -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" close.mp4
open close.mp4
~~~

# Camera animation in VisIt

Let's open our visualization in VisIt:

~~~{.bash}
cd ~/visualizeThis
visit -s positiveNegativePressure.py
~~~

This is the same script posted online but without saving the image to disk.

While the script is running, let's open Controls - Command... and paste the code:

~~~{.python}
c0 = GetView3D()   # save the view in this control point
print c0
~~~~

This will print out all attributes of the current view. We can also print them one-by-one:

~~~{.python}
print c0.focus
print c0.centerOfRotation
~~~~

In VisIt there is no analogue of ParaView's `GetActiveCamera().Azimuth` function -- instead you need to
set up several attributes simultaneously, depending on what you want to do. Even though c0.imageZoom =
c0.centerOfRotation, if we want to do rotation now, we'll have to figure out the angles (theta, phi) from
projections and then adjust several variables, and in such a way that the settings are compatible with
each other. We'll take a different route -- we'll reset the view first.

Let's reset the view:

~~~{.python}
from math import *
c1 = View3DAttributes()   # create a new view
phi = 0     # azimuthal angle 0 <= phi <= 2*pi
theta = 0   # vertical angle -pi/2 <= theta <= pi/2
c1.viewNormal = (cos(theta)*cos(phi),cos(theta)*sin(phi),sin(theta))
c1.focus, c1.viewUp = (0, 0, 0), (0, 0, 1)
c1.viewAngle, c1.parallelScale, c1.imageZoom = 30, 17.3205, 1
c1.nearPlane, c1.farPlane, c1.perspective = -34.641, 34.641, 1
c1.imageZoom = 3
c1.centerOfRotationSet = 1
SetView3D(c1)
~~~

Let's remove the axes and the bounding box:

~~~{.python}
a = AnnotationAttributes()
a.userInfoFlag = 0
a.axes3D.visible = 0
a.axes3D.bboxFlag = 0
SetAnnotationAttributes(a)
~~~

Set up the format and directory for saving frames:

~~~{.python}
s = SaveWindowAttributes()
s.format, s.family, s.outputToCurrentDirectory = s.PNG, 0, 0
s.outputDirectory = "/Users/razoumov/visualizeThis"
SetSaveWindowAttributes(s)
~~~

Let's elevate the camera slightly and then spin around the vertical axis:

~~~{.python}
nframes = 20
theta = pi/10.   # elevate the camera by 18 degrees
for i in range(nframes):
    phi = float(i)/float(nframes-1)*2.*pi/10.   # rotate from 0 to 36 degrees
    c1.viewNormal = (cos(theta)*cos(phi), cos(theta)*sin(phi), sin(theta))
    SetView3D(c1)
    s.fileName = 'step%04d'%(i)+'.png'
    SetSaveWindowAttributes(s)
    name = SaveWindow()
~~~
	
Merge these into a movie at 5 fps:

~~~{.bash}
ffmpeg -r 5 -i step%04d.png -c:v libx264 -pix_fmt yuv420p -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" spin.mp4
open spin.mp4
~~~

Let's fly into the visualization. We'll do that by moving the focal point away from us:

~~~{.python}
nframes = 20
focalStart = 0.
focalEnd = -35.
theta, phi = 0., 0.
c1.viewNormal = (cos(theta)*cos(phi),cos(theta)*sin(phi),sin(theta))
for i in range(nframes):
    x = focalStart + float(i)/float(nframes-1)*(focalEnd-focalStart)
    c1.focus = (x, 0., 0.)
    SetView3D(c1)
    s.fileName = 'step%04d'%(i)+'.png'
    SetSaveWindowAttributes(s)
    name = SaveWindow()
~~~

~~~{.bash}
ffmpeg -r 5 -i step%04d.png -c:v libx264 -pix_fmt yuv420p -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" approach.mp4
open approach.mp4
~~~

If you want to go into more advanced techniques, please check out our VisIt workshop materials at
http://bit.ly/2lYqypE. There we describe, e.g., how you can set up multiple control points and
interpolate smoothly between them, so that you combine several of these transitions in a single
animation.
