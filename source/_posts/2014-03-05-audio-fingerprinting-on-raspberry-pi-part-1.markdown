---
layout: post
title: "Audio fingerprinting on Raspberry Pi (part 1)"
date: 2014-03-05 18:15:43 -0500
comments: true
categories: [Audio, Python, Raspberry Pi]
---
A short guide on how to do some basic audio fingerprinting on a Raspberry Pi also known as “my quest to notify myself when the dryer is done”.
Lets start with the excellent Chromaprint library.
Build Chromaprint on Pi

Install pre-reqs:

    sudo apt-get install libboost1.50-all-dev ffmpeg libtag1-dev zlib1g-dev resample libresample1-dev cmake libffms2-dev  

Download package from Chromaprint source

    mkdir /opt/src
    cd /opt/src
    wget https://bitbucket.org/acoustid/chromaprint/downloads/chromaprint-1.1.tar.gz
    tar -zxvf chromaprint-1.1.tar.gz
    cd chromaprint\*
    cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_EXAMPLES=OFF -DCMAKE_CXX_FLAGS="-Ofast -mfpu=vfp -mfloat-abi=hard -march=armv6zk -mtune=arm1176jzf-s" -DBUILD_TOOLS=ON
    make
    sudo make install

Python stuff

If you don’t already have PIP installed go grab it:

    sudo easy_install pip

Build/install Python packages now:

    sudo pip install pyacoustid audioread

Create a test WAV file 10 seconds in length of whatever audio sound/noise/whatever that you want to fingerprint.
In this case I’m using my USB microphone plugged into the Pi since there is no onboard microphone or audio input (and no analog inputs either).
You can also use the PyAudio package you installed to record the wav file.

    arecord -D plughw:1,0 -d 10 -r 44100 --channels=2 --format=cd /tmp/test.wav

Fingerprint your wave file (in python). Note: You can just as easily use ‘fpcalc’ in the shell to get a compressed or raw fingerprint from chromaprint libs.

    import chromaprint
    import acoustid
    duration, fingerprint = acoustid.fingerprint_file('/tmp/test.wav')
    fp_raw = chromaprint.decode_fingerprint(fingerprint)[0]
    print "Compressed fingerprint: %s" % fingerprint
    print "Raw fingerprint: %s" % fp_raw

If all went well you should not have seen any errors and your compressed and raw fingerprint should have been printed to the screen.
If you fingerprint only had “AQAA” in it then something is wrong with your chromaprint library.
The available libchromaprint packages along with the python-pyaudio package did not work for me at all and consistently resulted in no fingerprints.
Instead building the custom chromaprint libs and then installing pyaudio via pip resulted in a working setup.

If you do have fingerprint errors make sure your wav file is not empty and if it’s not empty then try to do a clean build on the chromprint libs and re-install.

Note: The example ‘fpcalc’ won’t actually build from the 1.1 chromaprint libs. It throws some strange compiler errors that I couldn’t resolve.

I’m going to be recording 2 10 second files in a loop and that can be a bit hard on the SD card in the Raspberry Pi so instead I’m using the built in TMPFS location in /run so I don’t do a lot of unnecessary writes to the SD card.

That’s all for now, come back for part 2 where I wrap it all up into a single script that records in a thread while analyzing the previous recording and sending notifications when matches are found.

