# Kaldi

# Kaldi with GStreamer Plugin Deployment Guide for macOS ARM64

This guide provides step-by-step instructions for setting up Kaldi with GStreamer plugin support on macOS ARM64 architecture. Follow these instructions carefully to ensure a successful deployment.

## Prerequisites

First, create an isolated conda environment for the build:

```bash
conda create -n kaldi-gst python=3.10
conda activate kaldi-gst

```

## Install Required Dependencies

Install GStreamer and other required dependencies using Homebrew:

```bash
brew install gstreamer
brew install gstreamer-plugins-good
brew install gstreamer-tools
brew install glib
brew install pkg-config
brew install openblas
brew install portaudio
brew install automake autoconf libtool

```

## Set Up Environment Variables

Set necessary environment variables for the build process:

```bash
export KALDI_ROOT=$(pwd)
export OPENBLAS_ROOT=$(brew --prefix openblas)
export PKG_CONFIG_PATH="$(brew --prefix gstreamer)/lib/pkgconfig:$(brew --prefix glib)/lib/pkgconfig:$PKG_CONFIG_PATH"
export LDFLAGS="-L$(brew --prefix openblas)/lib $LDFLAGS"
export CPPFLAGS="-I$(brew --prefix openblas)/include $CPPFLAGS"
export PATH="$(brew --prefix pkg-config)/bin:$PATH"
export PATH="/opt/homebrew/opt/libtool/libexec/gnubin:$PATH"

```

## Handle Path Issues

If your path contains spaces, create a symlink to a path without spaces:

```bash
mv 'Application Support/AI2Apps/local/server/agents/AutoDeploy/projects/kaldi' ~/Library/kaldi_no_space_path
cd ~/Library/kaldi_no_space_path

```

## Build OpenFST

Download and build OpenFST (required by Kaldi):

```bash
cd tools
curl -L -o openfst-1.7.9.tar.gz https://www.openslr.org/resources/2/openfst-1.7.9.tar.gz
sed -i '' 's/OPENFST_VERSION ?= 1.8.4/OPENFST_VERSION ?= 1.7.9/' Makefile
make
cd ..

```

> Note: If you encounter any issues with OpenFST, make sure you're using version 1.7.9 which is known to work well with Kaldi.
> 

## Configure Kaldi

Configure Kaldi with shared library support (required for the GStreamer plugin):

```bash
cd src
./configure --shared --use-cuda=no --mathlib=OPENBLAS

```

> Troubleshooting: If you see an error about missing OpenFST, ensure you've built it correctly in the tools directory.
> 

## Install PortAudio

PortAudio is required for the online extensions:

```bash
cd ../tools
./extras/install_portaudio.sh
cd ../src

```

> Troubleshooting: If you encounter errors about missing portaudio.h, run the install_portaudio.sh script from the tools directory.
> 

## Fix OpenFST Compatibility Issue

If you encounter errors in the `openfst_compat.h` file during compilation, fix it with:

```bash
sed -i.bak 's/printer.Print(&os, s);/printer.Print(os, s);/g' fstext/openfst_compat.h

```

## Build Kaldi Core

Build the Kaldi core libraries:

```bash
make -j$(sysctl -n hw.ncpu)

```

> Note: This step can take 30-90 minutes depending on your system.
> 

## Build Kaldi Online Extensions

Build the online extensions needed for the GStreamer plugin:

```bash
make ext -j$(sysctl -n hw.ncpu)

```

> Troubleshooting: If you see errors about missing PortAudio, make sure you've run the install_portaudio.sh script.
> 

## Build the GStreamer Plugin

Build the Kaldi GStreamer plugin:

```bash
cd gst-plugin
make depend
make -j$(sysctl -n hw.ncpu)

```

## Validate Installation

Check that the GStreamer plugin was built successfully:

```bash
ls -lh libgstonlinegmmdecodefaster.so

```

Verify the Kaldi core build:

```bash
cd ../featbin
ls -lh compute-mfcc-feats

```

## Using the GStreamer Plugin

To use the GStreamer plugin, you may need to copy it to your GStreamer plugin path:

```bash
# Find your GStreamer plugin path
export GST_PLUGIN_PATH=$(pwd)/../gst-plugin

```

## Troubleshooting Common Issues

1. **Missing OpenFST**:
    - Error: `Could not find file /include/fst/fst.h`
    - Solution: Build OpenFST in the tools directory with `make`
2. **Missing PortAudio**:
    - Error: `portaudio.h file not found`
    - Solution: Run `../tools/extras/install_portaudio.sh`
3. **OpenFST Compatibility Issues**:
    - Error: `non-const lvalue reference to type 'std::ostream' cannot bind to a temporary`
    - Solution: Edit `fstext/openfst_compat.h` to fix the printer.Print call
4. **Path with Spaces**:
    - Error: Various build errors due to spaces in paths
    - Solution: Move Kaldi to a path without spaces
5. **Compiler Errors**:
    - Error: Various C++ compilation errors
    - Solution: Make sure you have the latest Xcode command-line tools installed

## Final Notes

- The GStreamer plugin examples can be found in `egs/voxforge/gst_demo`
- For real-time speech recognition, you'll need to set up a proper GStreamer pipeline
- If you need to use the plugin with your system GStreamer installation, copy the .so file to your GStreamer plugin path

This completes the installation of Kaldi with GStreamer plugin support on macOS ARM64.