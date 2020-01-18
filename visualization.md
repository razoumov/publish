This page http://bit.ly/teccvis

## Links

- User-facing visualization wiki https://docs.computecanada.ca/wiki/Visualization
- WestGrid's visualization training materials https://westgrid.github.io/trainingMaterials/tools/visualization
  - archive of 14 visualization webinars (since 2016)
- National visualization team https://wiki.computecanada.ca/staff/Visualization
  - meetings every third Monday of the month at 9am Pacific in TECC-Viz Vidyo room
  - meetings minutes posted on the wiki
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

- large-scale rendering workshops: included into ParaView slides, datasets, much prefer CPU rendering
- prefer client-server (interactive) or batch visualization
- no fans of X11 forwarding (too slow) or remote desktop (only if necessary)
- VNC https://docs.computecanada.ca/wiki/VNC (if client-server is not available)
  - `/cvmfs/soft.computecanada.ca/nix/var/nix/profiles/16.09/bin/vncserver` only on compute node (security!)
  - Graham's VDI Nodes
- X2Go server on selected systems (handles user authentication)

## Package installation
