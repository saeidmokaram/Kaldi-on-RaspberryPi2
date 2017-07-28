# Raspberry Pi2 Ubuntu MATE Kaldi compiling notes

## Installing required packages
Not all the packeges might be relivent to this installation
  
  sudo apt-get update
  
  sudo apt-get install git sox htop python-dev python-numpy python-scipy python-matplotlib python-pip python-pyaudio python3-pyaudio automake gf$
  
  sudo apt-get install bison mplayer c++11 libjack0 libjack-dev
  
  sudo apt-get install  zlib1g-dev

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
  make

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




# To activate swap on ubuntu mate after making swap partition do:
sudo swapon /dev/mmcblk0p3
