---
layout: post
title: "How to install Mayavi with Python 3 on Ubuntu 20.04 using pip or anaconda"
date: 2022-06-12

categories: tutorial technical-issue

---

Despite using mayavi for a long time, I still run into problem when reinstalling mayavi. Thus, I wrote this guide on how to install mayavi and solutions to some errors that I collected.

# Install mayavi using pip
1. Create conda environment to install mayavi with python 3.7 (I haven't tested other python version)
```
conda create -y -n mayavi python=3.7 
conda activate mayavi
```
2. Following the [official instruction](https://docs.enthought.com/mayavi/mayavi/installation.html) to install mayavi
```
pip install mayavi
pip install PyQt5
```
3. Set the correct ETS_TOOLKIT following the suggestion in [this comment](https://github.com/enthought/mayavi/issues/595#issuecomment-366534652)
```
export ETS_TOOLKIT=qt4
export QT_API=pyqt5
```
Optional: You might need to put these lines in ~/.bashrc and run `source ~/.bashrc`. These variables will be set automatically everytime you boot the computer.  
If you for get to set these variables ETS_TOOLKIT and QT_API, mayavi will return the following error:
```
Traceback (most recent call last):
  File "/home/acao/miniconda3/envs/mayavi/lib/python3.7/site-packages/tvtk/pyface/ui/qt4/scene.py", line 60, in resizeEvent
    self._scene._renwin.update_traits()
AttributeError: 'NoneType' object has no attribute 'update_traits'
Aborted (core dumped)
```

# Install mayavi using conda
You can install mayavi using **conda** as suggested in [this comment](https://github.com/cv-rits/MonoScene/issues/6#issuecomment-1009260023).
1. Install mayavi using conda:
```
conda install -c anaconda mayavi  
```
2. Downgrading numba:
```
pip install numba==0.53  
```
3. Downgrading TBB to version 2020.2:
```
conda install -c bioconda tbb=2020.2
```

# Collected errors
- `pip install mayavi`  fails at **building wheel for mayavi**. 
This error is pointed out in [this comment](https://github.com/cv-rits/MonoScene/issues/3#issuecomment-998662257). The reason is that the version of vtk is too new. Thus, the solution is to downgrade vtk version to 8.1.2:
```
pip install PyQt5  
pip install VTK==8.1.2
pip install mayavi
```

