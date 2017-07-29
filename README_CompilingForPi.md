![Kaldi Pi](https://raw.githubusercontent.com/saeidmokaram/Kaldi-on-RaspberryPi2/master/Kaldi_Pi.png)
# Raspberry Pi2 Ubuntu MATE Kaldi compiling notes
This is an instruction to compile the Kaldi ASR for a Raspberry Pi2/3 running [Ubuntu MATE for the Raspberry Pi 2/3](https://ubuntu-mate.org/raspberry-pi/).

## System requirements:

1. Raspberry Pi2/3
2. 32GB microSD card (Recommended). Full installation requires about 18GB.
3. Ubuntu MATE for the Raspberry Pi 2/3.
4. Allocating swap space to the Ubuntu MATE.
5. Booting Ubuntu MATE in the command mode.
6. Overclocking your Pi (Recommended)

## Preparing your Raspberry Pi2/3
The first step is to install and run the [Ubuntu MATE](https://ubuntu-mate.org/raspberry-pi/) on your Raspberry Pi 2/3.

The Kaldi installation requires a bit more memory than the 1GB of the Raspberry Pi 2/3. So an option for compiling Kaldi on the device is to allocate a swap space. The Ubuntu MATE 16.04.2 automatically resize the root partition on first boot to fully utilize the all available space on the microSD card. It doesn't allocate any swap space by default so we need to manually do that after the first boot of your MATE system. After making a swap space of about 2GB (by using 'Gparted' for instance), to activate swap on Ubuntu MATE do (you may need to modify the swap partition path):

    sudo swapon /dev/mmcblk0p3

To use less memory for you OS, you can set your system to boot in the command mode by modifying the system config in the 'Raspberry Pi Software Configuration Tool' -> 2. Boot Options:

    sudo raspi-config

Here you can also overclock your Pi (using a heat sink is recommended) in the '4. Overclock' option to:

    High 1000MHz ARM, 500MHz core, 500MHz SDRAM, 2 overvolt

## Installing required packages
Not all the packages might be relevant to this installation
  
    sudo apt-get update
    sudo apt-get install git sox htop python-dev python-numpy python-scipy python-matplotlib python-pip python-pyaudio python3-pyaudio automake gfortran subversion sshfs libtool flac swig kdevelop gnome-applets libatlas3-base gawk libatlas-base-dev libportaudio-ocaml jack nano openjdk-8-jdk portaudio19-dev bison mplayer c++11 libjack0 libjack-dev zlib1g-dev

## Getting Kaldi

    git clone https://github.com/kaldi-asr/kaldi.git kaldi --origin upstream


## Making required tools
Run your Pi2 in the "Shell Mode" and use just "one cpu" (-j 1) to compile because of memory limit on Pi2.

    cd kaldi/tools/

In Makefile, if required, uncomment the next line to build with OpenFst-1.4.1.

    OPENFST_VERSION = 1.4.1

Add this befor "all: check_required_programs sph2pipe atlas sclite openfst"

    CXXFLAGS = -mfpu=neon -march=native -mcpu=native -mtune=native -mfloat-abi=hard -funsafe-math-optimizations

Make the tools by:

    make -j 1

To install portaudio, modify install_portaudio.sh like:

    #if [ "$MACOS" != "" ]; then
      cp src/common/pa_ringbuffer.h install/include/
    #fi

Then install it by:

    ./install_portaudio.sh

## Making Kaldi:

    cd ../src

In src do:

    ./configure

Modify kaldi.mk by removing "-msse -msse2" in front of "CXXFLAGS = " and adding:

    -mfpu=neon -march=native -mcpu=native -mtune=native -mfloat-abi=hard -funsafe-math-optimizations

Modify the onlinebin/MakeFile like:

    EXTRA_LDLIBS += -lasound -lrt -ljack

Making depend:

    make depend -j 1
    make -j 1

You may need to do "make ext" to have onlinbin as well:

    make ext_depend -j 1
    make ext -j 1
