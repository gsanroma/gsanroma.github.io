---
title: 'offscreen rendering with pysurfer'
date: 2019-02-23
permalink: /posts/2019/01/pysurfer-offscreen-rendering/
tags:
  - neuroimaging
  - visualization
  - FreeSurfer
---

[Pysurfer](https://pysurfer.github.io/) is a Python package to display brain cortical surfaces with color overlays.
In its most common configuration, it needs a working graphics card and physical display to generate the graphics via OpenGL.
Therefore, you need to do some tweaking if you want to use in a remote (headless) server.
In this post, I explain how to set up offscreen rendering with Pysurfer.
A docker file reproducing all the steps in detail is available [here](/files/Dockerfile.cos7).

Pysurfer uses [Mayavi](https://docs.enthought.com/mayavi/mayavi/) to generate the graphics, which in turn relies on `vtk`.
Mayavi offers 3 possible solutions for offscreen rendering [here](https://docs.enthought.com/mayavi/mayavi/tips.html#off-screen-rendering).
Namely:
1. avoid the rendering window by setting `mlab.options.offscreen = True`
2. rendering using the virtual framebuffer
3. use `vtk` with `osmesa` for pure software rendering

Although options 1 and 2 are simpler, we did not manage to get them working.
This is why I explain here in detail the steps for option 3.
Specifically:
1. install `llvmpipe`, a library for software rendering
2. install `osmesa` and `glu` (with rendering via `llvmpipe`)
3. install `vtk` 
4. install `mayavi` and `pysurfer`

# Install `llvmpipe`

`llvmpipe` is a software renderer that will be used by `osmesa`.
Depending on the Linux distribution, there might be `osmesa_llvmpipe` package already available.
We did not manage to compile `vtk` using the distribution-specific `osmesa_llvmpipe` installation (as required in the 3rd step).
So we installed `llvmpipe` and compiled `osmesa` from source (in the next step).

You might need to install the following dependencies first (in centos):
```
yum install -q -y hdf5 hdf5-devel tcl tcl-devel tk tk-devel kk
```

Install `llvmpipe`:
```bash
yum install -y llvm-toolset-7
```

# Install `osmesa` and `glu`

Next download and compile `osmesa` and `glu` (OpenGL utility toolkit).
Both are later needed by VTK.

Download:
```bash
wget -q ftp://ftp.freedesktop.org/pub/mesa/older-versions/11.x/11.0.0/mesa-11.0.0-rc1.tar.gz
wget -q ftp://ftp.freedesktop.org/pub/mesa/glu/glu-9.0.0.tar.gz
```

Configure `osmesa` with the following options to enable software rendering:
```bash
./configure CXXFLAGS="-fPIC -O2 -g -DDEFAULT_SOFTWARE_DEPTH_BITS=31" CFLAGS="-fPIC -O2 -g -DDEFAULT_SOFTWARE_DEPTH_BITS=31" LDFLAGS="-lm -lstdc++" \
     --disable-xvmc --disable-glx --disable-dri --enable-opengl --disable-gles1 --disable-gles2 --disable-egl --with-dri-drivers="" --with-gallium-drivers="swrast" \
     --enable-texture-float --enable-shared-glapi --enable-gallium-osmesa --enable-gallium-llvm=yes --prefix=/opt/osmesa_llvmpipe
```
... and `make`, `make install`.

Configure `glu` with flags:
```bash
./configure PKG_CONFIG_PATH=/opt/osmesa_llvmpipe/lib/pkgconfig \
      CXXFLAGS="-fPIC -O2"    CFLAGS="-fPIC -O2"     LDFLAGS="-lm -lstdc++" \
      --enable-osmesa     --prefix=/opt/osmesa_llvmpipe \
```
... and `make`, `make install`.


# Compile `vtk`

Once we have correctly installed `osmesa` and `glu` (in `/opt/osmesa_llvmpipe`) we download and install `vtk` with software rendering via `osmesa`.


We found that VTK 5.8 worked as suggested by Patrick Snape in [his blog](https://patricksnape.github.io/2014/offscreen_rendering/).
Download:
```bash
wget -q http://www.vtk.org/files/release/5.8/vtk-5.8.0.tar.gz
```

We specify the directory where Mesa is installed:
```bash
MESA_INSTALL_PREFIX=/opt/osmesa_llvmpipe/
```

Then run `cmake` with the following options to build the Python wrappers and allow offscreen rendering:
```bash
cmake \
    -DBUILD_SHARED_LIBS=ON \
    -DVTK_WRAP_PYTHON=ON \
    -DVTK_USE_X=OFF \
    -DVTK_USE_SYSTEM_HDF5=ON \
    -DPYTHON_LIBRARY=/usr/lib64/libpython2.7.so \
    -DPYTHON_INCLUDE_DIR=/usr/include/python2.7/ \
    -DTCL_INCLUDE_PATH=/usr/include/tcl8.5/ \
    -DTK_INCLUDE_PATH=/usr/include/tk/ \
    -DOPENGL_INCLUDE_DIR=${MESA_INSTALL_PREFIX}/include \
    -DOPENGL_gl_LIBRARY=${MESA_INSTALL_PREFIX}/lib/libGLU.so \
    -DOPENGL_glu_LIBRARY=${MESA_INSTALL_PREFIX}/lib/libglapi.so \
    -DVTK_OPENGL_HAS_OSMESA=ON \
    -DOSMESA_INCLUDE_DIR=${MESA_INSTALL_PREFIX}/include \
    -DOSMESA_LIBRARY=${MESA_INSTALL_PREFIX}/lib/libOSMesa.so .. \
```

We next install the Python wrappers as follows:
```bash
cd /root/VTK/build/Wrapping/Python/
python setup.py install
```

# Install `mayavi` and `pysurfer`

Finally install Python packages normally:
```bash
pip install mayavi PySurfer==0.8.0
```

You should now be able to render graphics in pysurfer without need of a physical display.



