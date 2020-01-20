This page http://bit.ly/teccvis

## Links

- User-facing visualization wiki https://docs.computecanada.ca/wiki/Visualization
- WestGrid's visualization training materials https://westgrid.github.io/trainingMaterials/tools/visualization
  - archive of 14 visualization webinars (since 2016)
- National visualization team https://wiki.computecanada.ca/staff/Visualization
  - meetings every third Monday of the month at 9am Pacific in TECC-Viz Vidyo room
  - meetings minutes posted on the wiki https://wiki.computecanada.ca/staff/Visualization/Meeting_minutes
  - team's mailing list viz@computecanada.ca
  - user-facing front page http://bit.ly/cctopviz
  - CC Slack visualization channel https://computecanada.slack.com/archives/C0KEL4V5Z
- OTRS visualization queue
- Canada-wide fall *Visualize This!* competition https://computecanada.github.io/visualizeThis (last four years)
- *Seeing Big* showcase 2015-2017 (@HPCS, no longer running)

## Scalable, general-purpose 3D tools (training/documentation available)

* [ParaView full-day slides](https://westgrid.github.io/trainingMaterials/materials/paraviewWorkshop.pdf)
  (128 pages, since 2010, last updated December 2019)
  - introduction to sci-vis • ParaView architecture and GUI • importing data • filters • exporting
    scenes (need to add Cinema) • animation • scripting • remote visualization on CC clusters • more
    advanced topics in webinars

* [VisIt full-day slides](https://westgrid.github.io/trainingMaterials/materials/visitWorkshop.pdf) (129
  pages, since 2016, last updated May 2017)
  - introduction to sci-vis • VisIt architecture and GUI • importing data • operators • quantitative
    analysis • more controls: professional quality plots and animation • scripting • remote visualization
    on CC clusters • more advanced topics in webinars

* DHSI "3D visualization for the humanities" workshop (since 2016)

## Other tools (training/documentation available)

* 3D tools: VMD, VTK
* Volumetric plotting and analysis: yt
* Plotting: [Plotly](https://github.com/razoumov/publish/blob/master/plotly.md), Bokeh, Matplotlib, Gnuplot, Xmgrace
* Graphs: Gephi

## Future training

* VTK.js and other 3D web-based visualization (May-13 webinar)
* Cinema databases / Cinema Science https://github.com/cinemascience
* More of Python-based VTK
* VTK-m (C++ only)
* TTK (The Topology ToolKit, included into latest ParaView)
* VMD scripting
* In-situ visualization (Catalyst, LibSim)
* Open-source photogrammetry
* Open-source GIS
* Other advanced topics in ParaView and VisIt (e.g., Programmable Filter)

## Remote visualization on Compute Canada systems

- Large-scale rendering workshops: included into ParaView slides, datasets
- Much prefer CPU rendering (many reasons!)
- Recommend **client-server** (interactive data exploration or when need a GUI) or **batch** visualization
  (production work)
- Can use the Trace Tool to create a ParaView Python script and then debug it:
  # either locally on your laptop (small workflows) and then upload the script to a cluster, making sure to
    modify the paths, or
  # remotely via client-server (big workflows) and then upload the script
  - watch the webinar "Batch visualization on Compute Canada clusters" (2019-Sep-18)
- Do not use X11 forwarding (too slow)
  - if absolutely have to use it, ask users to re-enable INdirect GLX inside their X11 servers (disabled
  by default) to switch from cluster's CPU to laptop's GPU rendering, e.g. on a Mac:
```
defaults write org.macosforge.xquartz.X11 enable_iglx -bool true
>>> restart XQuartz
ssh -Y cedar.computecanada.ca
export LIBGL_ALWAYS_INDIRECT=1
glxspheres64   # now uses laptop's GPU ~55fps (up from ~2fps)
```
- No big fans of remote desktops either: use them only if necessary
  - VNC https://docs.computecanada.ca/wiki/VNC (if client-server is not available)
    - use `/cvmfs/soft.computecanada.ca/nix/var/nix/profiles/16.09/bin/vncserver` only on compute nodes
      (security!) together with SSH port forwarding
    - Graham's VDI Nodes
  - X2Go server on selected systems (handles user authentication)

## Packages

Three types of ParaView modules installed on CC clusters:

```
paraview-offscreen-gpu/5.4.1 (vis,D)    paraview-offscreen/5.5.2 (vis,D)    paraview/5.5.2 (vis,D)
```

Sometimes you will find that these do not include various 3rd-party tools, so you will need to
recompile. Recent example: needed a ParaView server with OSPRay rendering (bunch of dependencies) and
OSPRay materials built-in. In these cases compilation instructions can vary quite a bit -- please contact Alex.

The module `visit/2.13.3` includes both `./2.13.3/2.13.3/linux-x86_64/bin/engine_{ser,par}`, but you will
occasionally find limitations. The newer VisIt 3.x is not installed, as it proved to be very unstable on
the clusters. Debugging it is on the long to-do list.
