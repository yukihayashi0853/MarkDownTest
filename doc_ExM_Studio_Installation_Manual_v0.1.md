
# ExM Studio Installation Manual
# **Version Information**

| Date | Version | Author(s) |
| --- | --- | --- |
| 11/12/2018 | 0.1 | Atsushi Kajita, Robert Prior |

# **Key Terms and Abbreviations**

| Term | Description |
| --- | --- |
| ExM | Expansion Microscopy |
| PSF | Point Spread Function |
| GPU | Graphics Processing Unit |
| MATLAB | an integrated computing environment that combines numeric computation, advanced graphics and visualization, and a high-level programming language |
| FoV | Field of View |
| EPFL | ÉCOLE POLYTECHNIQUE FÉDÉRALE DE LAUSANNE |

# **Introduction**

This ExM Studio Installation Manual provides instructions on how to install and configure ExM Studio software on a server. Information about ExM Studio, including ExM Studio User's Manual, is on-line at

[http://exm.studio](http://exm.studio)

The first part of this manual is system requirements of hardware and software for ExM Studio. The second part describes general instructions on how to download ExM Studio packages, and external libraries and applications it needs, and install them.

# **System requirements**

This chapter describes system requirements for ExM Studio.

## **Hardware**

This table shows our recommended requirements of hardware for ExM Studio. We provide a server which satisfies the hardware requirements. Currently we support Pascal architecture of NVIDIA GPUs or newer.

| Processor | 1x Intel Xeon Gold 6128, 3.4GHz, 6Core |
| :---: | :--- |
| GPU | 1x NVIDIA RTX 2080 |
| RAM | 8x 16GB Memory |
| Storage | 1x M.2 512GB SSD (For OS) |
|         | 2x SATA 4TB HDD (For Data) |

The memory size depends on image size you process with ExM Studio. The stitching application takes an especially huge memory size. For example, supposing that one 2D tiff image is 10MB and the grid dimension of FoVs is 9x9 on xy plane and 100z−planes are processed, the expected maximum of memory usage is around 100GB at most.

## **Software**

This section describes supported OS, and required external libraries and applications.

### **Operating system**

ExM Studio currently runs on Linux OS only. CentOS 7.4 is verified to run ExM Studio.

### **External libraries and driver**

ExM deconvolution requires GPU-acceleration. NVIDIA provides a kernel driver and CUDA Toolkit for GPUs. CUDA Toolkit is a development environment for creating high performance GPU-accelerated applications. ExM Studio supports CUDA Toolkit 9.1 and 9.2.

Internally ExM deconvolution needs NVIDIA CUDA Deep Neural Network library (cuDNN) and NVIDIA Collective Communications Library (NCCL) which are also provided by NVIDIA. cuDNN is GPU-accelerated library of primitives for deep neural networks. NCCL is multi-GPU and multi-node collective communication primitives that are performance optimized for NVIDIA GPUs. They are already included in the deconvolution package.

| Libraries | Versions | URL |
| --- | --- | --- |
| CUDA Toolkit | 9.1, 9.2 | [https://developer.nvidia.com/cuda-toolkit-archive](https://developer.nvidia.com/cuda-toolkit-archive) |
| cuDNN | 7.3 | [https://developer.nvidia.com/cudnn](https://developer.nvidia.com/cudnn) |
| NCCL | 2.3.4 | [https://developer.nvidia.com/nccl](https://developer.nvidia.com/nccl) |
| NVIDIA driver | 390.46 | [https://www.nvidia.com/Download/index.aspx](https://www.nvidia.com/Download/index.aspx) |

### **Additional applications**

ExM deconvolution has two binaries: a command line and a MATLAB function. MATLAB is an integrated computing environment that combines numeric computation, advanced graphics and visualization, and a high-level programming language. If you use a MATLAB function, you need to install at least MATLAB R2018a that we verified to run our function.

Deconvolution process consists of two procedures: Point Spread Function (PSF) generation and image deconvolution with PSF. Currently we have not provided PSF generation function yet. We introduce an open source software for PSF generation, PSF Generator, which is developed by EPFL Biomedical Imaging Group ([http://bigwww.epfl.ch](http://bigwww.epfl.ch)).

| Applications | Versions | URL |
| --- | --- | --- |
| MATLAB | 2018a | [https://www.mathworks.com/products/matlab.html](https://www.mathworks.com/products/matlab.html) |
| PSF Generator | 18.12.2017 | [http://bigwww.epfl.ch/algorithms/psfgenerator/](http://bigwww.epfl.ch/algorithms/psfgenerator/) |





# **Installation**

Typically, installation of ExM Studio is simple. The first section describes how to install a software package for PSF generation, and ExM deconvolution binaries. The second section describes how to install TeraStitcher which is a free tool for fast automatic 3D-stitching of Teravoxel-sized tiled microscopy images.

ExM Studio uses the following directory structure.

This chapter describes each module functions, usage, and example in ExM Studio, supposing that ExM Studio package is located at application directory, 'AppDir'.

```
ExM-Studio/  
└─ deconv/                          # For ExM deconvolution package
|   └─ bin/  
|   |   |-exmDeconv                 # Command line
|   |   |-exmDeconv.mexa64          # Mex file for MATLAB
|   |   \-PSFGenerator.jar          # Jar file for PSF generation (not included)
|   ├─lib/
|   |   \-libexm_deconvolution.so   # Library of deconv engine |
|   \-config/
|       \-config.txt                # Configuration file for PSF Generator
\-stitching/                        # Folder to contain stitching package
```
    

## **Deconvolution**

This section describes how to install NVIDIA driver, CUDA toolkit library, software package for PSF generation, and ExM deconvolution.

### **NVIDIA driver**

NVIDIA driver is a kernel module for controlling GPUs. If you don't have NVIDIA driver, please install it by following these instructions.
1. Download an installation script for the latest driver at NVIDIA webpage
    1. Open NVIDIA DOWNLOAD DRIVERS page: [https://www.nvidia.com/Download/index.aspx](https://www.nvidia.com/Download/index.aspx) on web browser
    1. Select your own GPU type, for example, RTX-2080:
        1. Select 'GeForce' in a pull-down menu of 'Product Type'
        1. Select 'GeForce RTX 20 Series' in a pull-down menu of 'Product Series'
        1. Select 'GeForce RTX 2080' in a pull-down menu of 'Product'
        1. Select 'Linux 64-bit' in a pull-down menu of 'Operating System'
        1. Select 'English (US)' in a pull-down menu of 'Language'
        1. Click 'SEARCH' button
    1. Click 'DOWNLOAD' button, and open a new page for download
    1. Click 'DOWNLOAD' button
1. Stop X server if you use GUI on Linux 
 - `$ sudo init 3`
1. Run the installation script
 - `$ sudo sh NVIDIA-Linux-x86\_xxx.xx.run  (xxx.xx is driver version)`
1. Reboot a server 
 - `$ sudo reboot`

### **CUDA Toolkit**

CUDA Toolkit is provided by NVIDIA. ExM Studio runs with CUDA Toolkit 9.1 and 9.2. Here, CUDA Toolkit 9.1 installation is explained, as an example. If you have already installed other version of CUDA Toolkit, please note that the installed path is /usr/local/cuda or /usr/local/cuda-9.1.

1. Download an installation script at CUDA Toolkit webpage in NVIDIA as in the table of chapter 'System requirements'
    1. Open CUDA Toolkit archived web page: [https://developer.nvidia.com/cuda-91-download-archive](https://developer.nvidia.com/cuda-91-download-archive) on web browser
    1. Click 'Linux', 'x86\_64', 'CentOS', '7', and 'runfile (local)' buttons
    1. Click 'Download' button at Base Installer
1. Run an install script with sudo
 - `$ sudo sh cuda\_9.1.x\_xxx.xx\_linux.run  (xxx.xx is CUDA 9.1 revision)`
1. Configure shell environment variables in ~/.bash\_profile:
 - `export CUDA\_ROOT\_DIR=/usr/local/cuda`
 - `export PATH=$PATH:${CUDA\_ROOT\_DIR}/bin`
 - `export LD\_LIBRARY\_PATH=${LD\_LIBRARY\_PATH}:${CUDA\_ROOD\_DIR}/lib64`

More information on how to install CUDA Toolkit is here: [https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html)

### **NVIDIA cuDNN**

The NVIDIA cuDNN is a GPU-accelerated library for processing deep neural networks. Before installing cuDNN, you need to register for the NVIDIA Developer program: [https://developer.nvidia.com/accelerated-computing-developer](https://developer.nvidia.com/accelerated-computing-developer)

We verified cuDNN at least 7.3.

1. Download a tar file at cuDNN webpage in NVIDIA
    1. Open cuDNN home page: [https://developer.nvidia.com/cudnn](https://developer.nvidia.com/cudnn) on web browser
    1. Click 'Download cnDNN' button
    1. Complete the survey and click 'Submit' button
    1. Check a button for accepting the 'Terms and Conditions'
    1. Click 'Archived cuDNN Releases' link
    1. Click 'Download cuDNN v7.3.0 [Sept 19, 2018], for CUDA 9.0'
    1. Click 'cuDNN v7.3.0 Library for Linux' and get 'cudnn-9.0-linux-x64-v7.3.0.29.tgz'
1. Install the downloaded tar file
- `$ tar xvzf cudnn-9.0-linux-x64-v7.3.0.29.tgz`
- `$ sudo cp cuda/include/cudnn.h /usr/local/cuda/include`
- `$ sudo cp cuda/lib64/libcudnn\* /usr/local/cuda/lib64`
- `$ sudo chmod a+r /usr/local/cuda/include/cudnn.h`
- `$ sudo chmod a+r /usr/local/cuda/lib64/libcudnn\*`

### **NVIDIA NCCL**

The NVIDIA NCCL is a library of multi-GPU collective communications easily. Before installing NCCL, you need to register for the NVIDIA Developer program: [https://developer.nvidia.com/accelerated-computing-developer](https://developer.nvidia.com/accelerated-computing-developer)

We verified NCCL at least 2.3.4.

1. Download a tar file at NCCL webpage in NVIDIA
    1. Open cuDNN home page: [https://developer.nvidia.com/nccl](https://developer.nvidia.com/nccl) on web browser
    1. Click 'Download NCCL' button
    1. Complete the survey and click 'Submit' button
    1. Click 'NCCL legacy Downloads' for selecting the previous version
    1. Check a button for accepting the 'Terms and Conditions'
    1. Click 'Download NCCL v2.3.4, for CUDA 9.0, Sept 19, 2018'
    1. Click 'NCCL 2.3.4 O/S agnostic and CUDA 9.0' and get 'nccl\_2.3.4-1+cuda9.0\_x86\_64.txz'
1. Install the downloaded tar file
 - `$ tar xvJf nccl\_2.3.4-1+cuda9.0\_x86\_64.txz`
 - `$ sudo cp nccl\_2.3.4-1+cuda9.0\_x86\_64/include/nccl.h /usr/local/cuda/include`
 - `$ sudo cp nccl\_2.3.4-1+cuda9.0\_x86\_64/lib/libnccl.so\* /usr/local/cuda/lib64`
 - `$ sudo chmod a+r /usr/local/cuda/include/nccl.h`
 - `$ sudo chmod a+r /usr/local/cuda/lib64/libnccl.so\*`

### **Software package for PSF Generation**

PSF Generator is one of the most famous applications for generating PSF. The instructions of installation are below.

1. Download jar file of PSF Generator at [http://bigwww.epfl.ch/algorithms/psfgenerator/](http://bigwww.epfl.ch/algorithms/psfgenerator/)
1. Put the jar file into ExM-Studio/deconv/bin/
1. Download a sample configuration text file for PSF Generator at [http://bigwww.epfl.ch/algorithms/psfgenerator/config.txt](http://bigwww.epfl.ch/algorithms/psfgenerator/config.txt)
1. Put the config file into ExM-Studio/deconv/config/

### **ExM deconvolution**

ExM deconvolution package provides two ways of running deconvolution: a command line and MATLAB mex file. The instructions of installation are below.

1. Download a tar ball of ExM deconvolution at [http://exm.studio](http://exm.studio)
1. Untar the tar ball at the parent directory of ExM-Studio,
 - `$ cd AppDir`
 - `$ tar xzf ExM-deconvolution-ver0.1.tar.gz`
3. Put these environment variables into .bash\_profile
 - `export PATH=&quot;AppDir/ExM-Studio/deconv/bin&quot;:$PATH`
 - `export LD\_LIBRARY\_PATH=&quot;AppDir/ExM-Studio/deconv/lib&quot;:$LD\_LIBRARY\_PATH`

### **MATLAB**

For ExM deconvolution, MATLAB is able to be used for console to run. MATLAB can be installed following instructions at [https://www.mathworks.com/help/install/ug/choose-installation-procedure.html](https://www.mathworks.com/help/install/ug/choose-installation-procedure.html)

## **Stitching**

### **TeraStitcher**

Binary of TeraStitcher can be found at [https://github.com/abria/TeraStitcher/wiki/Binary-packages](https://github.com/abria/TeraStitcher/wiki/Binary-packages). There are options for Windows/Mac/Linux as well as two options between a command line only version with some GPU acceleration or a slightly older version that has a GUI. The CUDA accelerated command line version is recommended ([https://github.com/abria/TeraStitcher/wiki/Binary-packages#terastitcher-portable-command-line-version](https://github.com/abria/TeraStitcher/wiki/Binary-packages#terastitcher-portable-command-line-version)). Extract that package to ExM-Studio/stitching. For example, if ExM-Studio is in the home directory run:

 - `$ unzip -j -d ~/ExM-Studio/stitching/TeraStitcher-portable-1.11.6-Linux.zip`
