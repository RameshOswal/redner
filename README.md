# redner

redner is a differentiable Monte Carlo renderer that can take the derivatives of rendering output with respect to arbitrary 
scene parameters, that is, you can backpropagate from the image to your 3D scene. One of the major usages of redner is inverse rendering (hence the name redner) through gradient descent. A distinct feature of redner is that it is physically-based -- which means it simulates photons and produce realistic lighting phenomena, such as shadow and global illumination, and it handles the derivatives of these features correctly.

For more details on the renderer, what it can do, and the techniques it uses for computing the derivatives, please
take a look at the paper:
"Differentiable Monte Carlo Ray Tracing through Edge Sampling", Tzu-Mao Li, Miika Aittala, Fredo Durand, Jaakko Lehtinen
[https://people.csail.mit.edu/tzumao/diffrt/]
Since the submission we have improved the renderer a bit. In particular we implemented a CUDA backend and accelerated
the continuous derivatives significantly by replacing automatic differentiation with hand derivatives.

redner is expected to be used with [PyTorch](https://pytorch.org/), and can be used seamlessly with PyTorch operators.
While the documentation is work in progress, you can take a look at the [tests directory](tests) to have a basic sense.
A good starting point is to look at [tests/test_single_triangle.py](https://github.com/BachiLi/redner/blob/master/tests/test_single_triangle.py) where we optimize for the vertex positions of a single triangle.
redner inherits a subset of [Mitsuba](http://www.mitsuba-renderer.org) scene format,
see [tests/test_teapot_reflectance.py](https://github.com/BachiLi/redner/blob/master/tests/test_teapot_reflectance.py) and [tests/test_teapot_specular.py](https://github.com/BachiLi/redner/blob/master/tests/test_teapot_specular.py) for examples of loading Mitsuba scene files. There is also a Wavefront obj file loader for individual meshes, take a look at [pyredner/load_obj.py](https://github.com/BachiLi/redner/blob/master/pyredner/load_obj.py).

redner depends on a few libraries/systems:
- [Python 3.6 or above](https://www.python.org) (required)
- [pybind11](https://github.com/pybind/pybind11) (required)
- [PyTorch 0.4.1 or 1.0](https://pytorch.org) (required)
- [OpenEXR](https://github.com/openexr/openexr) (required)
- [OpenEXR Python](https://github.com/jamesbowman/openexrpython) (required, just clone the repo and python setup.py install)
- [Embree](https://embree.github.io) (required)
- [Thrust](https://thrust.github.io) (required, included in a submodule)
- [miniz](https://github.com/richgel999/miniz) (already in this repository)
- [CUDA 10](https://developer.nvidia.com/cuda-downloads) (optional)
- [optix prime](https://developer.nvidia.com/optix) (optional, required when compiled with CUDA)
- [miniconda](https://conda.io/miniconda.html) (optional, but recommended)
- A few other python packages: numpy, scikit-image

I recommend using conda to setup the Python related dependencies, e.g.:
```
conda install -c conda-forge pybind11
conda install pytorch-nightly -c pytorch
(for some reason Mac OS users have to use conda-forge's Python, see [https://github.com/pybind/pybind11/issues/1579]).
conda install -c conda-forge python
```

redner uses [CMake](https://cmake.org) as its build system. You need CMake 3.12 or above to build redner.
The build procedure follows common CMake instructions.
Ideally,
```
mkdir build
cd build
cmake ..
make install -j 8
```
should build and install the project, but you may need to tell CMake where the dependencies are by defining
the following variables:
```
Python_INCLUDE_DIRS
Python_LIBRARIES
EMBREE_INCLUDE_DIRS
EMBREE_LIBRARY
OptiX_INCLUDE
CUDA_LIBRARIES
THRUST_INCLUDE_DIR
optix_prime_LIBRARY
CUDA_curand_LIBRARY
```
I suggest using ccmake or other interfaces of cmake to setup the variables.

redner is tested under MacOS with clang 6.0 and Ubuntu with gcc 7.0. Windows is not tested yet but should be
portable with moderate modification. In general any compiler with c++14 support should work.

The current development plan is to enhance the renderer. Following features will be added in the near future (not listed in any particular order):
- Environment map
- Non-black background
- Stratification of random number
- More BSDFs e.g. glass/GGX
- A properer secondary edge sampling strategy 
  (to make the renderer computation friendlier to GPU, we temporarily dropped the hierarchical edge sampling algorithm described in the paper, and instead used an importance resampling strategy.
   see [edge.cpp](https://github.com/BachiLi/redner/blob/master/edge.cpp) )
- Support for edge shared by more than two triangles
  (The code currently assumes every triangle edge is shared by at most two triangles.
   If your mesh doesn't satisfy this, you can preprocess it in other mesh processing softwares such as [MeshLab](http://www.meshlab.net))
- Source-to-source automatic differentiation
- Mipmapping
- Russian roulette
- Distribution effects: depth of field/motion blur
- Proper pixel filter (currently only support 1x1 box filter)
- Mini-batching
- Documentation

If you have any questions/comments/bug reports, feel free to open a github issue or e-mail to the author
Tzu-Mao Li (tzumao@mit.edu)
