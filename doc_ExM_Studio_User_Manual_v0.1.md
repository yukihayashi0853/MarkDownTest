# ExM Studio User's Manual

# **Version Information**

| Date | Version | Author(s) |
| --- | --- | --- |
| 11/12/2018 | 0.1 | Atsushi Kajita, Robert Prior |

# **Key Terms and Abbreviations**

| Term | Description |
| --- | :--- |
| ExM | Expansion Microscopy |
| PSF | Point Spread Function |
| MATLAB | an integrated computing environment that combines numeric computation, advanced graphics and visualization, and a high-level programming language |
| FoV | Field of View |
| EPFL | ÉCOLE POLYTECHNIQUE FÉDÉRALE DE LAUSANNE |

# **Introduction**
This User's Manual is designed to provide documentation for people who will use ExM Studio. This manual describes ExM Studio functions and their usage. Installation Instructions are covered in another manual of 'ExM Studio Installation Manual'.
# **What is ExM Studio?**
Expansion Microscopy (ExM) Studio is a software package for Expansion Microscopy. Expansion Microscopy is a method for improving the resolution of light microscopy by physically expanding biological samples. Images of enlarged samples are captured many times to cover the entire specimen with a small FoV of the microscope. ExM Studio processes a lot of huge image data fast and effectively.

ExM Studio has two modules: deconvolution and stitching. Firstly, deconvolution is a mathematical operation for removing the out-of-focus information which is introduced by optical microscopy. With deconvolution, the microscopic image gets clearer. ExM Studio provides ExM deconvolution which is our GPU-accelerated deconvolution and optimized for NVIDIA GPU. Secondly, stitching is the process of combining multiple tiled images. Stitching takes a long time and requires lots of memory because it processes many images at the same time. For stitching a large number of images, currently, we introduce TeraStitcher, which is one of the promising software which has partial GPU acceleration.

ExM Studio uses the following directory structure.

This chapter describes each module functions, usage, and example in ExM Studio, supposing that ExM Studio package is located at application directory, 'AppDir'.

```
ExM-Studio/  
└─ deconv/                          # For ExM deconvolution package
|   └─ bin/  
|   |   ├─exmDeconv                 # Command line
|   |   ├─exmDeconv.mexa64          # Mex file for MATLAB
|   |   \─PSFGenerator.jar          # Jar file for PSF generation (not included)
|   ├─lib/
|   |   \─libexm_deconvolution.so   # Library of deconv engine |
|   \-config/
|       \-config.txt                # Configuration file for PSF Generator
\-stitching/                        # Folder to contain stitching package
```

## **Deconvolution**

Deconvolution process consists of two procedures: PSF generation and image deconvolution with the generated PSF. The following sections explain applications, functions, usage, and examples of PSF generation and ExM deconvolution.

### **PSF Generation**

Currently, we have not provided a PSF generation function yet. We introduce an open source software for PSF generation, PSF Generator, which is developed by EPFL Biomedical Imaging Group ([http://bigwww.epfl.ch](http://bigwww.epfl.ch)).

PSF Generator is a software package provided as a Java library. Here we explain how to run the PSF Generator jar file on MATLAB. PSF Generator has two options to generate PSF volume data: use GUI and use a configuration file.

In this version, ExM deconvolution only supports 3D uint16 data as input. Please note that '16-bits' is selected from a pull-down menu on the bottom of GUI dialog, or written as 'Type=16-bits' in a configuration file.

#### Usage:

| PSFGenerator |
| --- |
|     Package of PSF generation |
|   |
| **PSFGenerator.gui** |
|     Create GUI for PSF configuration. Parameters on GUI are stored in PSFGenerator |
|   |
| **psf = PSFGenerator.get** |
|     Create PSF Volume data using internally stored parameters    Output:        psf                                 3D matrix of PSF volume (uint16)  |
| **psf = PSFGenerator.compute(config\_filename)** |
|     Create PSF Volume data using parameters in a configuration file    Input:        Config\_filename          configuration filename for PSF Generator    Output:        psf                                  3D matrix of PSF volume (uint16) |



#### Example:

Using GUI on MATLAB:
```
$ matlab
>> javaaddpath('AppDir/ExM-Studio/deconv/bin/PSFGenerator.jar');
>> PSFGenerator.gui;
>> psf = PSFGenerator.get; 
```


\* Please note that output format of PSF volume should be '16-bits' currently.

Saving PSF data in TIFF format:

```
(Following the above)
>> saveastiff(psf,'psf.tif'); 
```

Note: 'saveastiff' function is provided at [https://www.mathworks.com/matlabcentral/fileexchange/35684-multipage-tiff-stack?focused=7519470&amp;tab=function](https://www.mathworks.com/matlabcentral/fileexchange/35684-multipage-tiff-stack?focused=7519470&amp;tab=function)

Using a configuration file on MATLAB:

```
$ matlab
>> javaaddpath('AppDir/ExM-Studio/deconv/bin/PSFGenerator.jar');
>> PSFGenerator.compute('config.txt'); 
```

### **ExM Deconvolution**

ExM deconvolution has two types of executables: a command line and a MATLAB mex file. We have implemented one of the classical algorithms for deconvolution, Richardson-Lucy deconvolution.

Before executing deconvolution, environment variables are set up in bash as below:

```
$ export PATH=&quot;AppDir/ExM-Studio/deconv/bin&quot;:$PATH
$ export LD\_LIBRARY\_PATH=&quot;AppDir/ExM-Studio/deconv/lib&quot;:$LD\_LIBRARY\_PATH |
```

These are able to be included in .bash\_profile according to your environment.

In this version, ExM deconvolution only supports 3D uint16 data of image and PSF volume as input. A command line of ExM deconvolution is currently able to load 3D TIFF images. MATLAB can load images in some additional formats like TIFF, HDF5, and so on.

#### Usage:

| exmDeconv img psf iters outputImg  (on shell) |
| --- |
|     A command line of ExM deconvolution |
|     Input:        img                                 Path to TIFF file of 3D image data (uint16) for deconvolution        psf                                  Path to TIFF file of PSF volume (uint16) applied        iters                                # of iterations in Richardson-Lucy deconvolution    Output:        outputImg                     Path of TIFF file (uint16) to save deconvolved image to  |
| deconv\_img = exmDeconv(img, psf, iters)   (on MATLAB) |
|     MATLAB mex file for deconvolution |
|     Input:        img                                 3D image data (uint16) for deconvolution        psf                                   PSF volume (uint16) applied        iters                                # of iterations in Richardson-Lucy deconvolution    Output:        deconv\_img                  deconvolved image (uint16)  |



#### Example:

Using a command line:

```
$ exmDeconv input.tif psf.tif 10 output.tif
```

Using a mex file on MATLAB:

```
$ matlab
>> addpath('AppDir/ExM-Studio/deconv/bin');
>> img = loadtiff('sample.tif');
>> psf = loadtiff('psf.tif');
>>
>> deconv\_img = exmDeconv(img, psf, 10);
```
Note: 'loadtiff' function is provided at [https://www.mathworks.com/matlabcentral/fileexchange/35684-multipage-tiff-stack?focused=7519470&amp;tab=function](https://www.mathworks.com/matlabcentral/fileexchange/35684-multipage-tiff-stack?focused=7519470&amp;tab=function)

## **Stitching**

The stitching module takes as input a volume consisting of a set of tiles each of which has a position and a set of images corresponding to depth. The stitching process combines these separate tiles into one large one where each image spans the entire acquisition target.

The guide is intended to act as a supplement to TeraStitcher's GitHub wiki located at [https://github.com/abria/TeraStitcher/wiki](https://github.com/abria/TeraStitcher/wiki). This guide contains an overview of the minimum required steps to perform the stitching but does not exhaustively cover all possible input parameters and options. For full possible run options please refer to TeraStitcher's wiki.

### **TeraStitcher**

#### Requirements:

To import data automatically the input data must be arranged in a two level directory hierarchy where the directory names have information about the mechanical displacement of the microscope between image captures. The root directory of the raw data should contain folders with 6 digit names. These represent the displacement in tenths of microns along the first dimension (which dimension can be specified in the input step). These directories each contain another set of directories representing displacement along the second dimension. They have names of 6 digits (same digits as parent directory) followed by an underscore followed by another 6 digits representing displacement in microns along the second dimension. Each of these folders then contain a set of TIFF files again with 6 digit names representing the depth displacement. For more information please see TeraStitcher's description at [https://github.com/abria/TeraStitcher/wiki/Supported-volume-formats#two-level-hierarchy-of-folders](https://github.com/abria/TeraStitcher/wiki/Supported-volume-formats#two-level-hierarchy-of-folders). It is possible to manually import data by producing a XML file describing the data however this is not recommended for most users.  Additionally, the user running TeraStitcher needs to have write permissions to this directory as some metadata is saved in the root directory of the input.

```
dataRoot/  
  |-069000/  
  |   |-069000\_151000/             
  |   |   |-145560.tif
  |   |   \-...
  |   \─069000\_154000/
  |       |-145560.tif
  |       |-...
  \-072000/
      |-072000\_151000/
      |   |-145560.tif
      |   \-...
      |-072000\_154000/
          |- 145560.tif 
          \-... 
```

Example Layout for a 2x2 Grid

#### Usage:

##### Overview:

TeraStitcher is broken into six steps, all of which run from the same executable. Command line arguments specify which step to run and each step produces a XML file to be used as input to the subsequent step (apart from the sixth which outputs the stitched volume). This guide will detail the minimum parameters required to perform stitching.  For a more detailed description of the steps and their possible parameters please refer to the command line interface guide at [https://github.com/abria/TeraStitcher/wiki/User-Interface#-command-line-interface-cli](https://github.com/abria/TeraStitcher/wiki/User-Interface#-command-line-interface-cli).

##### Common Parameters

- `--projin` all steps beside the first require a XML input produced by the previous step. The input file is specified with this parameter
- `--projout` all steps beside the sixth produce a XML file which, by default, is saved to the root of the input directory. The default location and name can be changed with this parameter. If this parameter is not specified, a file called xml\_NameOfOperation.xml will be created in the root of the input data directory.

##### Step 1: Import Volume

This step generates an XML file describing the input volume and contains information like the number of rows and columns the data has, where files are located on disk, the mechanical displacements between tiles, etc. The parameters are:

- Operation to run
  - `--import`
- Volume top level directory
  - `--volin`
- Reference system
  - Specifies what the levels in the folder hierarchy represent. Valid options are x, y, v, h as well as negative of each of these. For example `--ref1=y --ref2=-x` would specify the top level directory represents ascending rows and the inner directory represents descending columns. This would be equivalent to `--ref1=v --ref2=-h` (vertical and negative horizontal).
    - `--ref1`
    - `--ref2`
  - `--ref3` must be either d or z
- Voxel dimensions represent the size in microns along each dimension. This relates to field of view of the microscope.
  - `--vxl1`
  - `--vxl2`
  - `--vxl3`

This step also produces 2 files in the root of the input data folder:

- mdata.bin which contains additional binary data used by subsequent steps.
- test\_middle\_slize.tiff which contains an unstitched version of the middle depth of the input volume. This image is useful for verifying the correctness of the input parameters of this step.

##### Step 2: Displacement Computation
This step computes an initial displacement between tiles in the input. The step is a computationally heavy step as it requires reading every file in the input. The input parameters are:
- Operation to run
  - `--displcompute`
- Subvolume dimension
  - `--subvoldim` the number of images included in the displacement algorithm at a time. Effects runtime and accuracy of stitching. Value should be a large enough depth to cover of features in input; typical values are 100-200.

Additionally, this step, in the command line only version, has some GPU acceleration which is enabled by setting the environment variable `USECUDA\_X\_NCC=1.`

##### Step 3: Project Displacement
The initial computation of displacement produces a set of possible displacements. This refines them down to 1 displacement per set of tiles.
- Operation to run
  - `--displproj`
##### Step 4: Threshold Displacement
Each displacement has a reliability score in range [0 1]. This step replaces displacements that have a low reliability score with the mechanical displacements.
- Operation to run
  - `--displthres`
- Threshold which displacements must meet
  - `--threshold` good starting value is 0.7; affects accuracy of algorithm
##### Step 5: Place Tiles
Final displacement optimization; makes displacements globally consistent meaning any cycle of displacements will sum to 0. Currently uses the minimum spanning tree algorithm.
- Operation to run
  - `--placetiles`
##### Step 6: Merge tiles
Uses the optimized displacements to generate a stitched volume. The stitched volume is contained in a folder named RES(XxYxZ) where X Y and Z are the dimensions of the stitched volume. The folder hierarchy similar to what is expected as input. However, the output only has a single row and column directory. The innermost directory contains a set of large TIFF files which are slices across the entire volume iterating over the depth. This step also loads all input images making it a computationally expensive step.
- Operation to run
  - `--merge`
- Volume output directory
  - `--volout`
Outputs:
- A set of TIFF files representing the volume are saved to a folder located in volout folder.

#### Example:

Assuming the TeraStitcher binary folder is in the home directory, as is a volume in a folder called dataRoot the following commands show an example run-through using minimum parameters.
```
$ ~/ExM-studio/stitching/terastitcher --import --volin=&quot;$HOME/dataRoot&quot; --ref1=y --ref2=x --ref3=z --vxl1=0.7 --vxl2=0.7 --vxl3=1 --volin\_plugin=&quot;TiledXY|3Dseries&quot; 
$ USECUDA\_X\_NCC=1 ~/ExM-studio/stitching/terastitcher --displcompute --projin=&quot;$HOME/dataRoot/xml\_import.xml&quot;
$ ~/ExM-studio/stitching/terastitcher --displproj --projin=&quot;$HOME/dataRoot/xml\_displcomp.xml&quot;
$ ~/ExM-studio/stitching/terastitcher --displthres --projin=&quot;$HOME/dataRoot/xml\_displproj.xml&quot; --threshold=0.7
$ ~/ExM-studio/stitching/terastitcher --placetiles --projin=&quot;$HOME/dataRoot/xml\_displthres.xml&quot;
$ ~/ExM-studio/stitching/terastitcher --merge --projin=&quot;$HOME/dataRoot/xml\_merging.xml&quot; --volout=&quot;$HOME&
```

In this example the input data has a top level directory representing rows and an inner directory representing columns (hence `--ref1=y --ref2=x`). In the inner folders, there are multipage TIFF files for images across depth. These commands create XML files in the root of the input data directory which are used by the subsequent intermediary step (passed using `--projin`). In the final step a folder RES(XxYxZ) is created in the home directory containing a set of TIFFs which are large stitched slices of the input (X, Y, Z are dimensions specific to input data / parameters).


# **Execution Examples**

This section goes over a complete run through of ExM Studio where raw data from a microscope is first cleaned with deconvolution and then passed to TeraStitcher to create large stitched and cleaned images. Please see the previous sections for more details on individual steps.

The first step is to create a PSF TIFF file specific to the particular microscope which was used to capture the data. As the PSF is an approximation of noise of the microscope, having one PSF tailored to a specific microscope improves the result of deconvolution. See section [PSF Generation](https://docs.google.com/document/d/1kqR1inTcKLkxEk0RtTcRITSQ0qJVhFyRhfekPbIEWfw/edit#bookmark=id.h0wczjpmnf3w) for more details.

Assuming a PSF file has been created the next step is to run deconvolution. Assuming the PSF file is in current directory, the raw input images are in a directory called raw, and the output directory is setup according to what is required for the stitching process (see section [Requirements](https://docs.google.com/document/d/1kqR1inTcKLkxEk0RtTcRITSQ0qJVhFyRhfekPbIEWfw/edit#bookmark=id.k7g0clkjqrf)), the deconvolution commands can run as follows:

```
$ exmDeconv raw/1.tif psf.tif 10 ~/dataRoot/069000/069000\_151000/145560.tif 
$ exmDeconv raw/2.tif psf.tif 10 ~/dataRoot/069000/069000\_154000/145560.tif
$ exmDeconv raw/3.tif psf.tif 10 ~/dataRoot/075000/075000\_154000/145560.tif
$ exmDeconv raw/4.tif psf.tif 10 ~/dataRoot/075000/075000\_151000/145560.tif
$ exmDeconv raw/5.tif psf.tif 10 ~/dataRoot/072000/072000\_154000/145560.tif$ exmDeconv raw/6.tif psf.tif 10 ~/dataRoot/072000/072000\_151000/145560.tif |
```

Theses cleaned images can now directly be used as input to TeraStitcher. Below shows the minimal set of parameters needed to run the stitching. Note that these depend heavily on the input data. See section [Usage](https://docs.google.com/document/d/1kqR1inTcKLkxEk0RtTcRITSQ0qJVhFyRhfekPbIEWfw/edit#bookmark=id.bew9qjweerf0) for a description of the parameters.
```
$ ~/ExM-studio/stitching/terastitcher --import --volin=&quot;$HOME/dataRoot&quot; --ref1=y --ref2=x --ref3=z --vxl1=0.7 --vxl2=0.7 --vxl3=1 --volin\_plugin=&quot;TiledXY|3Dseries&quot; 
$ USECUDA\_X\_NCC=1 ~/ExM-studio/stitching/terastitcher --displcompute --projin=&quot;$HOME/dataRoot/xml\_import.xml&quot;
$ ~/ExM-studio/stitching/terastitcher --displproj --projin=&quot;$HOME/dataRoot/xml\_displcomp.xml&quot;
$ ~/ExM-studio/stitching/terastitcher --displthres --projin=&quot;$HOME/dataRoot/xml\_displproj.xml&quot; --threshold=0.7
$ ~/ExM-studio/stitching/terastitcher --placetiles --projin=&quot;$HOME/dataRoot/xml\_displthres.xml&quot;$ ~/ExM-studio/stitching/terastitcher --merge --projin=&quot;$HOME/dataRoot/xml\_merging.xml&quot; --volout=&quot;$HOME&quot; |
```
This will place XML files in the dataRoot directory which contain information about the stitching process. and will generate a folder in the home directory containing the large stitched volume.
