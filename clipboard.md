```bash
cd ~/visualizeThis
/bin/rm -rf {spin,approach,final}.mp4 frame00*.png
paraview --state=bladesWithLines.pvsm
```


https://github.com/razoumov/publish/raw/master/visit.zip



Now let's open ParaView's Python shell.

help(GetActiveCamera)
dir(GetActiveCamera())
help(GetActiveCamera().Azimuth)

# normally, this is how you would rotate it around the vertical axis:
camera = GetActiveCamera()
camera.Azimuth(10)
Render() or SaveScreenshot
# however, should not rotate now as CenterOfRotation is off-focus

v1 = GetActiveView()
print v1.CameraFocalPoint
print v1.CameraParallelScale
print v1.CameraPosition
print v1.CameraViewAngle
print v1.CameraViewUp
print v1.CenterOfRotation

[0.228557134725041, 2.47726112917046, 0.658093579072617]
4.25832870332
[-14.0557805400554, 9.23241645115292, 5.24329881316648]
30.0
[0.25357815545749796, -0.11566287189265197, 0.9603750408774258]
[1.8223249912262, -0.010200023651123, 0.0]

ResetCamera()    # reset the camera to include the entire scene but preserve orientation;
		 # this will rescale the picture and put CenterOfRotation into CameraFocalPoint

print v1.CameraFocalPoint
print v1.CameraParallelScale
print v1.CameraPosition
print v1.CameraViewAngle
print v1.CameraViewUp
print v1.CenterOfRotation

[1.7140189409255981, 10.003308773040771, 0.7980973720550537]
25.6202838711
[-84.22786381056636, 50.64577885934507, 28.38503766931171]
30.0
[0.25357815545749796, -0.11566287189265197, 0.9603750408774258]
[1.7140189409255981, 10.003308773040771, 0.7980973720550537]

# now we could do rotation, but probably we don't want to as the picture is somewhat small;
# let's move the camera a little bit closer to (CenterOfRotation = CameraFocalPoint)

v1.CameraPosition = [(0.6*a + 0.4*b) for a, b in zip(v1.CameraPosition,v1.CameraFocalPoint)]
Render()

# now let's do rotation
camera = GetActiveCamera()
camera.Azimuth(10)
Render() or SaveScreenshot

# but we want to save each frame
nframes = 50
for i in range(nframes):
    print v1.CameraPosition
    camera.Azimuth(360./nframes)
    SaveScreenshot('/Users/razoumov/visualizeThis/frame%04d'%(i)+'.png')

ffmpeg -r 10 -i frame%04d.png -c:v libx264 -pix_fmt yuv420p -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" spin.mp4
open spin.mp4

ResetCamera()

# let's fly towards the focal point
initialCameraPosition = v1.CameraPosition[:]   # force a real copy, not a pointer a = b
nframes = 50
for i in range(nframes):
    coef = float(i+0.5)/float(3*nframes)  # runs from 0 to 1/3
    print coef, v1.CameraPosition
    v1.CameraPosition = [((1.-coef)*a + coef*b) for a, b in zip(initialCameraPosition,v1.CameraFocalPoint)]
    SaveScreenshot('/Users/razoumov/visualizeThis/frame%04d'%(i)+'.png')

ffmpeg -r 10 -i frame%04d.png -c:v libx264 -pix_fmt yuv420p -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" approach.mp4
open approach.mp4

# now, let's attempt to move the focal point and the center of rotation towards the turbine blades

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

ffmpeg -r 10 -i frame%04d.png -c:v libx264 -pix_fmt yuv420p -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" final.mp4
open final.mp4
