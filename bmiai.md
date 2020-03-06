<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Introduction](#introduction)
  - [Cluster environment](#cluster-environment)
  - [Python on Cedar](#python-on-cedar)
- [GPU practical example (TensorFlow) on CC clusters](#gpu-practical-example-tensorflow-on-cc-clusters)
- [MATLAB on CC clusters](#matlab-on-cc-clusters)
  - [Running codes using the license at runtime](#running-codes-using-the-license-at-runtime)
    - [Serial code](#serial-code)
    - [Parallel code](#parallel-code)
  - [Compiling codes with the license and then running without the license](#compiling-codes-with-the-license-and-then-running-without-the-license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Introduction

Interactive hands-on workshop for BMIAI (Biomedical Imaging and Artificial Intelligence Research Cluster)
@UBC (2020-Mar-09).

* Overview of CC systems available and potential uses: computing, GPU, storage
* Sockeye system
* Explanation of accounts and the CCDB

## Cluster environment

1. First steps: logging in and transferring files with `scp`, `sftp` and Globus
1. Do not run anything CPU-intensive on login the nodes.
1. Use `salloc` for interactive testing and debugging and `sbatch` for production batch jobs. **You need to
   know how to switch from `salloc` to `sbatch` in all examples below.**
1. Sometimes adding `--partition=cpubase_interac` to your `salloc` command might help.

## Python on Cedar

- Identifying what modules are needed for a given Python code
- Determining if these are already available on CC
- Building an environment to install needed modules
- Submitting jobs making use of the created environment





# GPU practical example (TensorFlow) on CC clusters

Training a model from BMIAI, in "10 mins or less" on Cedar, code from Adam. First, we install the packages:

```
$ module load python/3.7.4
$ virtualenv --no-download ~/brain    # create virtual environment
$ source ~/brain/bin/activate         # load this environment
$ pip install --no-index --upgrade pip
$ pip install --no-index numpy
$ avail_wheels --name "*tensorflow_gpu*" --all_versions  # check out the available packages
$ pip install --no-index tensorflow_gpu==1.14.1   # voxelmorph relies on some TensorFlow 1.x features (tf.Dimension)
$ pip install --no-index Keras
$ pip install --no-index nibabel tqdm Pillow matplotlib
```

Next, download VoxelMorph and some data files:

```
$ cd ~/scratch
$ git clone https://github.com/voxelmorph/voxelmorph.git
$ mkdir bmiai && cd bmiai
$ mkdir -p input/learn2reg-mri-3d && cd input/learn2reg-mri-3d
$ download learn2reg-mri-3d.zip from https://www.kaggle.com/adalca/learn2reg-mri-3d/data # requires registration
$ mkdir ../learn2reg-unsupervised-models && cd ../learn2reg-unsupervised-models
$ wget http://bit.ly/39vUhtP -O cvpr2018_vm2_cc.h5
```

Submit a serial interactive job:

```
$ cd ~/scratch/bmiai
$ salloc --account=def-razoumov-ac --gres=gpu:1 --cpus-per-task=6 --mem-per-cpu=3600 --time=0:60:0
$ source ~/brain/bin/activate    # load your Python environment, only if you have not done so before
$ module load cuda cudnn
$ python
```

Inside the shell you are running the following Python code:

```
import os, sys
import numpy as np
import keras.layers   # ignore the warnings about 'libnvinfer* and some TensorRT libraries

sys.path.append('/scratch/razoumov/voxelmorph/')
sys.path.append('/scratch/razoumov/voxelmorph/src/')
sys.path.append('/scratch/razoumov/voxelmorph/ext/neuron/')
sys.path.append('/scratch/razoumov/voxelmorph/ext/pynd-lib/')
sys.path.append('/scratch/razoumov/voxelmorph/ext/pytools-lib/')

import networks
vol_shape = [160, 192, 224]   # our data will be of shape 160 x 192 x 224
ndims = 3
nb_enc_features = [16, 32, 32, 32]
nb_dec_features = [32, 32, 32, 32, 32, 16, 16]
unet = networks.unet_core(vol_shape, nb_enc_features, nb_dec_features);
disp_tensor = keras.layers.Conv3D(ndims, kernel_size=3, padding='same', name='disp')(unet.output)

import neuron
spatial_transformer = neuron.layers.SpatialTransformer(name='image_warping')
moved_image_tensor = spatial_transformer([unet.inputs[0], disp_tensor])
vxm_model = keras.models.Model(unet.inputs, [moved_image_tensor, disp_tensor])
val_volume_1 = np.load('/scratch/razoumov/bmiai/input/learn2reg-mri-3d/subject_1_vol.npz')['vol_data']
seg_volume_1 = np.load('/scratch/razoumov/bmiai/input/learn2reg-mri-3d/subject_1_seg.npz')['vol_data']
val_volume_2 = np.load('/scratch/razoumov/bmiai/input/learn2reg-mri-3d/atlas_norm_3d.npz')['vol']
seg_volume_2 = np.load('/scratch/razoumov/bmiai/input/learn2reg-mri-3d/atlas_norm_3d.npz')['seg']
val_input = [val_volume_1[np.newaxis, ..., np.newaxis], val_volume_2[np.newaxis, ..., np.newaxis]]
vxm_model.load_weights('/scratch/razoumov/bmiai/input/learn2reg-unsupervised-models/cvpr2018_vm2_cc.h5')
val_pred = vxm_model.predict(val_input)    # on Cedar takes few seconds

moved_pred = val_pred[0].squeeze()
pred_warp = val_pred[1]
mid_slices_fixed = [np.take(val_volume_2, vol_shape[d]//2, axis=d) for d in range(ndims)]
mid_slices_fixed[1] = np.rot90(mid_slices_fixed[1], 1)
mid_slices_fixed[2] = np.rot90(mid_slices_fixed[2], -1)
mid_slices_pred = [np.take(moved_pred, vol_shape[d]//2, axis=d) for d in range(ndims)]
mid_slices_pred[1] = np.rot90(mid_slices_pred[1], 1)
mid_slices_pred[2] = np.rot90(mid_slices_pred[2], -1)
import matplotlib as mpl
mpl.use('Agg')     # switch to PNG backend; could also use PS or PDF backends
neuron.plot.slices(mid_slices_fixed + mid_slices_pred, cmaps=['gray'], do_colorbars=True, grid=[2,3]); # no window
neuron.plot.plt.savefig('tmp1.png')

warp_model = networks.nn_trf(vol_shape)
warped_seg = warp_model.predict([seg_volume_1[np.newaxis,...,np.newaxis], pred_warp])

from pytools import plotting as pytools_plot

[ccmap, scrambled_cmap] = pytools_plot.jitter(255, nargout=2)
scrambled_cmap[0, :] = np.array([0, 0, 0, 1])
ccmap = mpl.colors.ListedColormap(scrambled_cmap)

mid_slices_fixed = [np.take(seg_volume_1, vol_shape[d]//1.8, axis=d) for d in range(ndims)]
mid_slices_fixed[1] = np.rot90(mid_slices_fixed[1], 1)
mid_slices_fixed[2] = np.rot90(mid_slices_fixed[2], -1)

mid_slices_pred = [np.take(warped_seg.squeeze(), vol_shape[d]//1.8, axis=d) for d in range(ndims)]
mid_slices_pred[1] = np.rot90(mid_slices_pred[1], 1)
mid_slices_pred[2] = np.rot90(mid_slices_pred[2], -1)

slices = mid_slices_fixed + mid_slices_pred
for si, slc  in enumerate(slices):
    slices[si][0] = 255

neuron.plot.slices(slices, cmaps = [ccmap], grid=[2,3]); # opens a matplotlib window
neuron.plot.plt.savefig('tmp2.png')
```

When done, quit Python, quit the interactive job and download 'tmp1.png' and 'tmp2.png' to your laptop.








# MATLAB on CC clusters

* Cedar has a universal license for all users
* Graham / Béluga use institutional licenses, so you'll need to write a file ~/.licenses/matlab.lic:

```
# MATLAB license passcode file
SERVER <ip address> ANY <port>
USE_SERVER
```

```
$ module avail matlab
matlab/2014a (t)   matlab/2016b (t)   matlab/2017a (t)   matlab/2018a (L,t)
matlab/2018b (t,D)   matlab/2019a (t)   matlab/2019b (t)
```

All examples below are on Cedar.

## Running codes using the license at runtime

### Serial code

Store the following serial code as `cosplot.m` in `~/scratch/bmiai`:

```
function cosplot()
nterms = 5;
fourbypi = 4.0/pi;
np = 100;
y(1:np) = pi/2.0;
x(1:np) = linspace(-2.0*pi,2*pi,np)

for k = 1:nterms
 twokm = 2*k-1;
 y = y - fourbypi*cos(twokm*x)/twokm^2;
end

plot(x,y)
print -dpsc matlab_test_plot.ps
quit
```

```
$ cd ~/scratch/bmiai
$ salloc --time=0:30:0 --mem-per-cpu=3600 --account=...
$ module load matlab/2018a    # will check the license
$ matlab -nodisplay -singleCompThread -r "cosplot"
```

All standard output from the last command will go to the terminal. It will also write its runtime
environment (host/OS/CPU/flags) into `/tmp/java.log.XXXXX`, and lots of miscellaneous stuff into
`~/.matlab/local_cluster_jobs/`.

### Parallel code

Store the following parallel code as `parfor.m` in `~/scratch/bmiai`:

```
p = parpool(2)      # create MATLAB parallel pool
tic                 # start timer
parfor idx = 1:10
  pause(2)
end
toc                 # print elapsed time since `tic`
p.delete()
quit
```

```
$ cd ~/scratch/bmiai
$ salloc --time=0:30:0 --cpus-per-task=2 --mem-per-cpu=3600 --account=...
$ module load matlab/2018a
$ srun matlab -nodisplay -r "parallel"    #  here `srun` is optional, but it's a good practice to still use it
```

We can also run parallel code interactively in the shel, e.g. for the parallel code:

```
$ cd ~/scratch/bmiai
$ salloc --time=0:30:0 --cpus-per-task=2 --mem-per-cpu=3600 --account=...
$ module load matlab/2018a
$ matlab -nodesktop   # this starts MATLAB shell
>> maxNumCompThreads   % returns 2
>> p = parpool(2)
>> tic
>> parfor idx = 1:10
>> pause(2)
>> end
>> toc     % will print elapsed time since `tic`
>> p.delete()
>> quit
```

## Compiling codes with the license and then running without the license

`mcc` compiler is provided in versions 2014a, 2018a and later

```
$ cd ~/scratch/bmiai
$ module load matlab/2018a
$ mcc -m -R -nodisplay cosplot.m     # compile serial code (takes a while ...), will also create /tmp/java.log.XXXXX
```

Once compiled, you can run the code without the license. You have two options:

(1) either manually setting your environment

```
$ salloc --time=0:30:0 --mem-per-cpu=3600 --account=...
$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$EBROOTMATLAB/bin/glnxa64:$EBROOTMATLAB/runtime/glnxa64
$ ./cosplot
```

(2) or the official way

```
$ salloc --time=0:30:0 --mem-per-cpu=3600 --account=...
$ module load mcr/R2018a
$ setrpaths.sh --path cosplot   # do this once for each compiled binary
run_mcr_binary.sh cosplot
```

Compiling and running a parallel code is similar:

```
$ cd ~/scratch/bmiai
$ module load matlab/2018a
$ mcc -m -R -nodisplay parallel.m    # compile parallel code (takes 3-5 minutes ...)
$ salloc --time=0:30:0 --mem-per-cpu=3600 --account=...
$ module load mcr/R2018a
$ setrpaths.sh --path parallel       # do this once for each compiled binary
$ srun run_mcr_binary.sh parallel    # here srun is necessary
```
