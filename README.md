#Anaconda recipes for OpenSesame
This repository contains recipes for the [conda-build](http://conda.pydata.org/docs/building/build.html) tool which can be used to create anaconda packages of the libraries on which [OpenSesame](https://github.com/smathot/OpenSesame) depends. For now, these packages will be stored in the [Anaconda Cloud's CogSci channel](https://anaconda.org/CogSci). This repository and manual are mainly meant for the maintainers of these recipes/packages and do not really have a purpose for the public.

##Installing the packages built from these recipes
These packages should be readily available in Anaconda for installation from the CogSci channel. For instance, to install OpenSesame in an Anaconda environment, it should be sufficient to issue the command

    conda install -c cogsci python-opensesame

This will install OpenSesame plus the basic dependencies it requires to run. It does not include optional dependencies, or libraries required to make the experiment backends function. In the *Environments* section you find instructions for installing OpenSesame in full.

##Building a recipe
First, make sure you are familiar with the basics of the conda-build tool by reading its documentation: http://conda.pydata.org/docs/building/build.html. The basic workflow for building these recipes is as follows.

1. Make sure you have conda-build installed in your Anaconda environment.

    `conda install conda-build`

2. Go to the folder in which these recipes are located and build a package by invoking

    `conda-build <package-name>/<ver> -c cogsci`

    where the -c flag indicates that the cogsci channel should be searched for additional dependencies (e.g. packages from this recipes repository that already have been built)

3. The above command only builds the package for the currently active python version. If you want to create builds for other python versions, you will have to add the `--python <PY_VER>` flag, for example, to build for python 3.4 the command is.

    `conda-build <package-name>/<ver> -c cogsci --python 3.4`

    or to build for all python versions:

    `conda-build <package-name>/<ver> -c cogsci --python all`

    After the build has been completed it will be saved at

    `path/to/anaconda/conda-bld/<your-platform>`

    for instance, if anaconda is installed in you homedir, and you have just built the python-opensesame package for python 3.5, you will find your build at

    `~/anaconda/conda-bld/osx-64/python-opensesame-3.1.0-py35_0.tar.bz2`

    (where of course the version number 3.1.0 can differ depending on which version you are packaging)

4. If your packages only contain python code and assets (and do not depend on acompanying compiled c-libraries), you can convert these packages for other platforms by using the `conda convert` command. For instance, if you would like to convert all python-opensesame packages for all platforms, you can issue the command

    `conda convert python-opensesame-3.1.0-py* -p all -o ~/build/python-opensesame`

    The `-p` flag indicates the platform for which the current package should be converted (e.g. win-32, linux-64) which in this example is all platforms.  
    The `-o` flag specifies an output folder in which the converted packages should be placed. If the -o flag is ommitted, the output folder will be the current working directory.

    After the process is completed, you can navigate to the output folder with `cd ~/build/python-opensesame`, you will see the folders linux-32, linux-64, osx-64, win-32 and win-64. Each of these folders will contain packages for each python version.

6. You can now choose to upload these packages to Anaconda cloud. Make sure you are logged in by issuing `anaconda login`. You can then upload all these packages to the cogsci channel at once by issuing

    `anaconda upload */* -u cogsci`

    if you omit the `-u cogsci` flag the package will be uploaded to your own, single-user, account. Optionally, cou also add the --label flag to add a label to the upload. For instance, of you upload an alpha or beta version you could add

    `anaconda upload */* -u cogsci --label alpha`

7. Once this is done, your package should now be available for download and installation from the anaconda cloud, with the command 

    `conda install -c cogsci <package_name>`


**Note:** Instead of appending the `-c cogsci` flag to most commands, you can also choose to search in the cogsci channel by default by adding it to the *.condarc* file that can be found in your home folder (or you can create one there). For more information about this, see http://conda.pydata.org/docs/config.html#the-conda-configuration-file-condarc. For instance, a basic .condarc file that also searches in the cogsci channel could look like this:

     channels:
        - cogsci
        - defaults

Don't forget to also add the `- defaults` option, otherwise anaconda will no longer search in its own channels.

## Environments

The `envs` directory contains yaml-files that describe Anaconda environments which can be used for running OpenSesame in. Some yaml files are very specific, in that they list the version number and python version for each package. This may be useful for releases, in which OpenSesame is always bundled with specific versions of its dependencies. 

The `opensesame_all.yaml` file is the most general description of an environment in which OpenSesame can run; it only lists the minimum of required dependencies without their version numbers or a python version with which a fully functional OpenSesame installation can run. Using this environment will automatically install the newest version of each available package.

### Using these environments
All environments that are found here are stored at http://anaconda.org/CogSci. 
The `conda env` command does not allow the addition of extra channels to search yet (the -c flag in the other commands), so for the following commands to work, you are required to at the CogSci channel to your *.condrc* file.

Environments can simply be imported in your Anaconda installation by issuing:

    conda env create cogsci/<environment_file> <your_optional_environment_name>

For example, to install the general OpenSesame environment, you can issue:

    conda env create cogsci/opensesame_all

**Note:** On Windows, the 'nomkl' package may give some trouble as it is not available for that platform yet (which is quite weird as it itself is a 'fake' package to prevent the much larger MKL versions of numpy and scipy to be installed). If this is the case, it can be safely removed from the list.

Of course, it's also possible to install OpenSesame without using environments (for instance, if you just want to have everything available in your root environment and don't want to use virtual environments). To do so, you will have to issue the following commands in Anaconda (the `-c cogsci` flag is ommitted in the following commands, but is required if cogsci is not in your default channels list in *.condarc*):

    conda install python-opensesame pygame pyglet pyopengl pyopengl-accelerate
    conda install ipython jupyter pillow
    pip install python-bidi expyriment

### Exporting an environment

Environments can be exported by issuing the `conda env export > <yaml_file>` command while *the environment to be exported is activated*. For instance

    conda env export > opensesame.yaml

will export a description of the current environment to the `opensesame.yaml` file. This file can be uploaded to anaconda by issuing

    conda env upload --file=opensesame.yaml <online_env_name>

It is only possible to upload to your own account at the moment, and sadly not to the CogSci organization's account. You will have to transfer the ownership of the uploaded yaml file manually on anaconda.org
