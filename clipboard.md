You can find a copy of these notes at http://bit.ly/chAnimation.

# Camera animation in ParaView

Let's open our visualization from the ParaView state file:

~~~{.bash}
cd ~/visualizeThis
/bin/rm -rf {spin,approach,final}.mp4 frame00*.png
paraview --state=bladesWithLines.pvsm
~~~

Now let's open ParaView's Python shell:

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
    camera.Azimuth(360./nframes)
    SaveScreenshot('/Users/razoumov/visualizeThis/frame%04d'%(i)+'.png')
~~~

We can merge the frames into a Quicktime-compatible MP4 movie at 10 fps:

~~~{.bash}}
ffmpeg -r 10 -i frame%04d.png -c:v libx264 -pix_fmt yuv420p -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" spin.mp4
open spin.mp4
~~~

Let's reset the camera and try to fly 1/3 of the way towards the focal point:

~~~{.python}
ResetCamera()
initialCameraPosition = v1.CameraPosition[:]   # force a real copy, not a pointer a = b
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

For the final animation, let's reset the camera, move the focal point and the center of rotation 80% of
the way towards the turbine blades, and then spin around the blades:

~~~{.python}
ResetCamera()
v1.CameraFocalPoint[1] = v1.CameraFocalPoint[1] - 10   # as the blades are centered towards positive y
v1.CenterOfRotation = v1.CameraFocalPoint[:]
v1.CameraPosition = [(0.2*a + 0.8*b) for a, b in zip(v1.CameraPosition,v1.CameraFocalPoint)]
Render()
camera = GetActiveCamera()
nframes = 50
for i in range(nframes):
    camera.Azimuth(360./nframes)
    SaveScreenshot('/Users/razoumov/visualizeThis/frame%04d'%(i)+'.png')
~~~

Let's make the final movie:

~~~{.bash}
ffmpeg -r 10 -i frame%04d.png -c:v libx264 -pix_fmt yuv420p -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" final.mp4
open final.mp4
~~~

# Camera animation in VisIt
