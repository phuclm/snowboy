# Snowboy Hotword Detection

by [KITT.AI](http://kitt.ai).

[Home Page](https://snowboy.kitt.ai)

[Full Documentation](http://docs.kitt.ai/snowboy)

[Discussion Group](https://groups.google.com/a/kitt.ai/forum/#!forum/snowboy-discussion) (or send email to snowboy-discussion@kitt.ai)

(The discussion group is new since September 2016 as we are getting many messages every day. Please send general questions there. For bugs, use Github issues.)

Version: 1.1.0 (9/20/2016)

## New!

Snowboy now offers **Hotword as a Service** through the ``https://snowboy.kitt.ai/api/v1/train/``
endpoint. Check out the [Full Documentation](http://docs.kitt.ai/snowboy) and example [Python/Bash script](examples/REST_API) (other language contributions are very welcome).

## Introduction

Snowboy is a customizable hotword detection engine for you to create your own
hotword like "OK Google" or "Alexa". It is powered by deep neural networks and
has the following properties:

* **highly customizable**: you can freely define your own magic phrase here –
let it be “open sesame”, “garage door open”, or “hello dreamhouse”, you name it.

* **always listening** but protects your privacy: Snowboy does not use Internet
and does *not* stream your voice to the cloud.

* light-weight and **embedded**: it even runs on a Raspberry Pi and consumes
less than 10% CPU on the weakest Pi (single-core 700MHz ARMv6).

* Apache licensed!

Currently Snowboy supports:

* all versions of Raspberry Pi (with Raspbian based on Debian Jessie 8.0)
* 64bit Mac OS X
* 64bit Ubuntu (12.04 and 14.04)
* iOS
* Android

It ships in the form of a **C++ library** with language-dependent wrappers
generated by SWIG. We welcome wrappers for new languages -- feel free to send a
pull request!

If you want support on other hardware/OS, please send your request to
[snowboy@kitt.ai](mailto:snowboy.kitt.ai)

## Precompiled node module

Snowboy is available in the form of a native node module precompiled for:
64 bit Ubuntu, MacOS X, and the Raspberry Pi (Raspbian 8.0+). For quick
installation run:

    npm install --save snowboy

For sample usage see the `examples/Node` folder. You may have to install
dependencies like `fs`, `wav` or `node-record-lpcm16` depending on which script
you use.

## Precompiled Binaries with Python Demo
* 64 bit Ubuntu [12.04](https://s3-us-west-2.amazonaws.com/snowboy/snowboy-releases/ubuntu1204-x86_64-1.1.0.tar.bz2)
  / [14.04](https://s3-us-west-2.amazonaws.com/snowboy/snowboy-releases/ubuntu1404-x86_64-1.1.0.tar.bz2)
* [MacOS X](https://s3-us-west-2.amazonaws.com/snowboy/snowboy-releases/osx-x86_64-1.1.0.tar.bz2)
* Raspberry Pi with Raspbian 8.0, all versions
  ([1/2/3/Zero](https://s3-us-west-2.amazonaws.com/snowboy/snowboy-releases/rpi-arm-raspbian-8.0-1.1.0.tar.bz2))
  
If you want to compile a version against your own environment/language, read on.

## Dependencies

Snowboy's Python wrapper uses PortAudio to access your device's microphone.

### Mac OS X

`brew` install `swig`, `sox`, `portaudio` and its Python binding `pyaudio`:

    brew install swig portaudio sox
    pip install pyaudio

If you don't have Homebrew installed, please download it [here](http://brew.sh/). If you don't have `pip`, you can install it [here](https://pip.pypa.io/en/stable/installing/).

Make sure that you can record audio with your microphone:

    rec t.wav

### Ubuntu/Raspberry Pi

First `apt-get` install `swig`, `sox`, `portaudio` and its Python binding `pyaudio`:

    sudo apt-get install swig3.0 python-pyaudio python3-pyaudio sox
    pip install pyaudio
    
Then install the `atlas` matrix computing library:

    sudo apt-get install libatlas-base-dev
    
Make sure that you can record audio with your microphone:

    rec t.wav
        
If you need extra setup on your audio (especially on a Raspberry Pi), please see the [full documentation](http://docs.kitt.ai/snowboy).

## Compile a Node addon
Compiling a node addon for Linux and the Raspberry Pi requires the installation of the following dependencies:

    sudo apt-get install libmagic-dev libatlas-base-dev

Then to compile the addon run the following from the root of the snowboy repository:

    node-pre-gyp clean configure build

## Compile a Python Wrapper

    cd swig/Python
    make

SWIG will generate a `_snowboydetect.so` file and a simple (but hard-to-read) python wrapper `snowboydetect.py`. We have provided a higher level python wrapper `snowboydecoder.py` on top of that.
    
Feel free to adapt the `Makefile` in `swig/Python` to your own system's setting if you cannot `make` it.


## Compile an iOS Wrapper

Using Snowboy library in Objective-C does not really require a wrapper. It is basically the same as using C++ library in Objective-C. We have compiled a "fat" static library for iOS devices, see the library here `lib/ios/libsnowboy-detect.a`.

To initialize Snowboy detector in Objective-C:

    snowboy::SnowboyDetect* snowboyDetector = new snowboy::SnowboyDetect(
        std::string([[[NSBundle mainBundle]pathForResource:@"common" ofType:@"res"] UTF8String]),
        std::string([[[NSBundle mainBundle]pathForResource:@"snowboy" ofType:@"umdl"] UTF8String]));
    snowboyDetector->SetSensitivity("0.45");        // Sensitivity for each hotword
    snowboyDetector->SetAudioGain(2.0);             // Audio gain for detection

To run hotword detection in Objective-C:

    int result = snowboyDetector->RunDetection(buffer[0], bufferSize);  // buffer[0] is a float array

You may want to play with the frequency of the calls to `RunDetection()`, which controls the CPU usage and the detection latency.

## Compile an Android Wrapper

    cd swig/Android
    # Make sure you set up the NDKROOT variable in Makefile before you run.
    # We have only tested with NDK version r11c.
    make

Using Snowboy library on Android devices is a little bit tricky. We have only tested with NDK version r11c. We do not support r12 yet because of the removal of armeabi-v7a-hard ABI in r12. We have compiled Snowboy using Android's cross-compilation toolchain for ARMV7 architecture, see the library here `lib/android/armv7a/libsnowboy-detect.a`. We then use SWIG to generate the Java wrapper, and use Android's cross-compilation toolchain to generate the corresponding JNI libraries. After running `make`, two directories will be created: `java` and `jniLibs`. Copy these two directories to your Android app directory (e.g., `app/src/main/`) and you should be able to call Snowboy funcitons within Java.

To initialize Snowboy detector in Java:

    # Assume you put the model related files under /sdcard/snowboy/
    SnowboyDetect snowboyDetector = new SnowboyDetect("/sdcard/snowboy/common.res",
                                                      "/sdcard/snowboy/snowboy.umdl");
    snowboyDetector.SetSensitivity("0.45");         // Sensitivity for each hotword
    snowboyDetector.SetAudioGain(2.0);              // Audio gain for detection

To run hotword detection in Java:

    int result = snowboyDetector.RunDetection(buffer, buffer.length);   // buffer is a short array.

You may want to play with the frequency of the calls to `RunDetection()`, which controls the CPU usage and the detection latency.

## Quick Start for Python Demo

Go to the `examples/Python` folder and open your python console:

    In [1]: import snowboydecoder
    
    In [2]: def detected_callback():
       ....:     print "hotword detected"
       ....:
    
    In [3]: detector = snowboydecoder.HotwordDetector("resources/snowboy.umdl", sensitivity=0.5, audio_gain=1)
    
    In [4]: detector.start(detected_callback)
    
Then speak "snowboy" to your microphone to see whetheer Snowboy detects you.

The `snowboy.umdl` file is a "universal" model that detect different people speaking "snowboy". If you want other hotwords, please go to [snowboy.kitt.ai](https://snowboy.kitt.ai) to record, train and downloand your own personal model (a `.pmdl` file).

When `sensitiviy` is higher, the hotword gets more easily triggered. But you might get more false alarms.

`audio_gain` controls whether to increase (>1) or decrease (<1) input volume.

Two demo files `demo.py` and `demo2.py` are provided to show more usages.

Note: if you see the following error:

    TypeError: __init__() got an unexpected keyword argument 'model_str'
    
You are probably using an old version of SWIG. Please upgrade. We have tested with SWIG version 3.0.7 and 3.0.8.

## Advanced Usages & Demos

See [Full Documentation](http://docs.kitt.ai/snowboy).

## Change Log

**9/28/2016**

* Offering Hotword as a Service through ``/api/v1/train`` endpoint.
* No version bump since decoder is not changed.

**v1.1.0, 9/20/2016**

* Added library for Node.
* Added support for Python3.
* Added universal model `alexa.umdl`
* Updated universal model `snowboy.umdl` so that it works in noisy environment.

**v1.0.4, 7/13/2016**

* Updated universal `snowboy.umdl` model to make it more robust.
* Various improvements to speed up the detection.
* Bug fixes.

**v1.0.3, 6/4/2016**

* Updated universal `snowboy.umdl` model to make it more robust in non-speech environment.
* Fixed bug when using float as input data.
* Added library support for Android ARMV7 architecture.
* Added library for iOS.

**v1.0.2, 5/24/2016**

* Updated universal `snowboy.umdl` model
* added C++ examples, docs will come in next release.

**v1.0.1, 5/16/2016**

* VAD now returns -2 on silence, -1 on error, 0 on voice and >0 on triggered models
* added static library for Raspberry Pi in case people want to compile themselves instead of using the binary version

**v1.0.0, 5/10/2016**

* initial release
