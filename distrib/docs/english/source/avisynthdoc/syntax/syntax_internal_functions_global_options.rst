
AviSynth Syntax - Global options and resource control
=====================================================

Memory, CPU, cache directory and other global settings adjustments.

SetMemoryMax
~~~~~~~~~~~~
::

    SetMemoryMax(int amount, int "type", int "index")

Sets the maximum memory (in MB) that AviSynth uses for its internal Video
Frame cache to the value of *amount*. Setting to zero just returns the current Memory Max value.

In Avisynth+ default Memory Max is 1024MB for 32 bits and 4096MB on the x64 version.
This limit is fine-tuned however if physical memory is low:

``DefaultMemoryMax = minimum(physical_memory / 4, secondary_memory_max_limit)``

``Type`` and ``index`` are additional arguments for devices such as GPUs.
Used to be used in Avisynth Neo, where Avisynth core was able to govern CUDA aware plugins
and their resources: memory usage could be managed individually for devices such as GPUs.

This special option was introduced in Neo, but is not used in ordinary Avisynth release.

.. describe:: int amount

    Device (including CPU) memory limit (MB) 

.. describe:: int type

    Device type. The following values are available:

    * ``DEV_TYPE_CPU``: CPU (default) 
    * ``DEV_TYPE_CUDA``: GPU 

.. describe:: int index

    Device number. Same as onCUDA device_index. Only 0 for DEV_TYPE_CPU. 

    Default value: 0 

Return value: Actual MemoryMax value set.

*Examples:*
::

    SetMemoryMax(16384)


SetCacheMode
~~~~~~~~~~~~
::

    SetCacheMode(int mode)

Fine tunes the internal video frame caching strategy.

Available values:

*   0 or ``CACHE_FAST_START``: start up time and size balanced mode (default)
*   1 or ``CACHE_OPTIMAL_SIZE`` slow start up but optimal speed and cache size 

SetMaxCPU
~~~~~~~~~
::

    SetMaxCPU([string feature1, string feature2, ...])

A debug control method. Intel processor specific.

Limits the CPU capabilities which AviSynth reports to its core and thus for external plugins and filters
through :ref:`env->GetCPUFlags <cplusplus_getcpuflags>`.

Available values:

*   ``""`` or ``"none"`` for zero SIMD support, no processor flags are reported
*   ``"mmx"``, ``"sse"``, ``"sse2"``, ``"sse3"``, ``"ssse3"``, ``"sse4"`` or ``"sse4.1"``,
    ``"sse4.2"``, ``"avx"``, ``"avx2"`` 

Parameters are case insensitive. 

Note: ``"avx2"`` triggers FMA3 flag as well. 

* Processor options w/o any modifier will limit the CPU flag report to at most the processor level.
* When "feature" is ended by '+', relevant processor feature flag will be switched on
* When "feature" is ended by '-', relevant processor feature flag will be removed. 

Multiple options can be put in a comma separated list. They will evaluate in that order. 

*Examples:*
::

    SetMaxCPU("SSE2") #reports at most SSE2 processor (even if AVX2 is available)
    SetMaxCPU("avx,sse4.1-") #limits to avx2 but explicitely removes reporting sse4.1 support
    SetMaxCPU("none,avx2+") #limits to plain C, then switches on AVX2-only support

OnCPU
~~~~~

OnCPU/OnCUDA (collectively called OnDevice)

Since 3.6 Avisynth Neo features were backported to Avisynth+. 
Such as supporting "devices", like CPU and CUDA and the data transfer between them.

Neo has rewritten some internal and external plugins to CUDA processing.
CUDA support needs special build, still experimental after v3.7. 
Avisynth itself does not process in CUDA, even not in such builds, just
provides the framework work such plugins.

If all are valid, the chain will be as follows.

Upstream → Upstream cache → Thread → Transfer → Downstream cache → Downstream → is 
the flow of frame data (reverse of GetFrame call direction)

Number of prefetch frames

*   0: Synchronous call without all cache
*   1: Synchronous call, but only transfer is read ahead and executed asynchronously. Downstream cache is enabled.
*   2 or more: Pre-read upstream processing using threads. Both upstream and downstream caches are valid. 

The number of upstream threads is fixed at 1 thread when prefetch = 2 or more, and 
the number of prefetches is fixed at 2. The downstream look-ahead number is set to the specified prefetch sheet.

::

    OnCPU(clip, int "num_prefetch") 

.. describe:: clip

    This clip is processed by the CPU. In other words, the processing before this is processed by the CPU. 

.. describe:: int num_prefetch

    Here you specify the number of frames to prefetch. About 2 will give you enough performance. 
    Unlike Prefetch, it has only one thread because it is a prefetch for parallelizing processing on the GPU and CPU. 

    default: 0 

If 0 is specified, it will be a synchronous call without using threads.

OnCUDA
~~~~~~
::

    OnCUDA(clip, int "num_prefetch", int "device_index")

.. describe:: clip

    This clip is processed by CUDA. In other words, the processing before this is processed by CUDA.
    A filter that does not support CUDA processing will result in an error. (answering a specific ScriptEnvironment request)

    Currently, internal filters are rarely (=not) supported, so you can only use external filters that are specially made.

.. describe:: int  num_prefetch =

    Same as OnCPU prefetch. Here you specify the number of frames to prefetch. About 2 will give you enough performance.
    Unlike Prefetch, it has only one thread because it is a prefetch for parallelizing processing on the GPU and CPU. 

    default: 0 

.. describe:: int  device_index =

    Specifies the GPU to run. If you have only one GPU, you can only use 0. 
    If you have two GPUs, you can specify 0 or 1. There is no limit on the number. 

    default: 0 


Of course, valid only on Avisynth+ built with CUDA option and works if the system has proper device and driver combination. 

SetWorkingDir
~~~~~~~~~~~~~
::

    SetWorkingDir(path)

Sets the default directory for AviSynth to the *path* argument. This is
primarily for easy loading of source clips, :doc:`importing <../corefilters/import>` scripts, etc. It
does not affect plugins' autoloading.

Return value is 0 if successful, -1 otherwise.

*Examples:*
::

    SetWorkingDir("c:\my_presets")
    AviSource("border_mask.avi")  # this loads c:\my_presets\border_mask.avi

SetPlanarLegacyAlignment
~~~~~~~~~~~~~~~~~~~~~~~~
::

    SetPlanarLegacyAlignment(mode)

Set alignment mode for `planar`_ frames. *mode* can either be true or false.
Some older (?pre 2005?) :doc:`plugins <../externalplugins>` illegally assume the layout of video frames in memory.
This special filter forces the memory layout of planar frames to be
compatible with prior versions of AviSynth. The filter works on the
GetFrame() call stack, so it effects filters **before** it in the script.

*Examples:*
::

    Example - Using an older version of Mpeg2Source() (1.10 or older):

    LoadPlugin("...\Mpeg2Decode.dll")
    Mpeg2Source("test.d2v")         # A plugin that illegally assumes the layout of memory
    SetPlanarLegacyAlignment(true)  # Set legacy memory alignment for prior statements
    ConvertToYUY2()     # Statements through to the end of the script have
    ...                             # advanced memory alignment.

Global variables OPT_xx
~~~~~~~~~~~~~~~~~~~~~~~

OPT_AllowFloatAudio
-------------------
::

    global OPT_AllowFloatAudio = true ## default false

Float audio is converted to 16 bit when frameserving through ACM, unless ``OPT_AllowFloatAudio``
is set to true (this option enables ``WAVE_FORMAT_IEEE_FLOAT`` audio output). 
In that case the audio is kept as it is. When accessing AviSynth directly (like MeGUI, BeHappy 
or ffmpeg do for example), there is no automatic conversion. 

The automatic conversion is done for clients that cannot handle Float audio (in the old days 
most of them couldn't).

Note: conversion takes place after the script processing is finished. Float audio is always allowed 
within the script.


OPT_UseWaveExtensible
---------------------
::

    global OPT_UseWaveExtensible = true ## default false

This option enables ``WAVE_FORMAT_EXTENSIBLE`` audio output. The default is
``WAVE_FORMAT_EX``.

*Note:*

Note: The default DirectShow component for .AVS files, "AVI/WAV File Source", 
does not correctly implement WAVE_FORMAT_EXTENSIBLE processing, so many application may not be 
able to detect the audio track. There are third party DirectShow readers that do work correctly. 
Intermediate work files written using the AVIFile interface for later DirectShow processing 
will work correctly if they use the DirectShow "File Source (async)" component or equivalent. 


OPT_dwChannelMask
-----------------
::

    global OPT_dwChannelMask(int v)   v2.60 

This option enables you to set ChannelMask.
It overrides WAVEFORMATEXTENSIBLE.dwChannelMask which is set according to this table:

When using these OPT, only VfW clients are affected, but not others such as ffmpeg.
Since Avisynth+ 3.7.3 audio channel masks are part of the system.

::

    0x00004, // 1   -- -- Cf
    0x00003, // 2   Lf Rf
    0x00007, // 3   Lf Rf Cf
    0x00033, // 4   Lf Rf -- -- Lr Rr
    0x00037, // 5   Lf Rf Cf -- Lr Rr
    0x0003F, // 5.1 Lf Rf Cf Sw Lr Rr
    0x0013F, // 6.1 Lf Rf Cf Sw Lr Rr -- -- Cr
    0x0063F, // 7.1 Lf Rf Cf Sw Lr Rr -- -- -- Ls Rs

OPT_AVIPadScanlines
-------------------
::

    global OPT_AVIPadScanlines = true ## default false   v2.60 

This option enables DWORD aligned planar padding. Default is packed aligned planar padding.
See memory alignment used in the AVIFile output emulation.

http://avisynth.nl/index.php/AVIFile_output_emulation

OPT_VDubPlanarHack
------------------
::

    global OPT_VDubPlanarHack = true ## default false   v2.60 

This option enables flipped YV24 and YV16 chroma planes. This is an hack for
early versions of Virtualdub with YV24/YV16 support.

OPT_Enable_V210
---------------
::

    global OPT_Enable_V210 = true ## default false   AVS+ 

When enabled, for 10bit YUV422, frameserve interleaved V210 instead of planar P210. (VfW) 

VfW here means Video For Windows clients such as VirtualDub.

When using these OPTs, only VfW clients are affected, but not others such as ffmpeg.


OPT_Enable_Y3_10_10
-------------------
::

    global OPT_Enable_Y3_10_10 = true ## default false   AVS+ 

When enabled, for 10bit YUV422, set the FourCC to ``Y3[10][10]`` ('Y', '3', 10, 10) instead of P210 ('P', '2', '1', '0'). (VfW)

OPT_Enable_Y3_10_16
-------------------
::

    global OPT_Enable_Y3_10_16 = true ## default false   AVS+ 

When enabled, for 16bit YUV422 ``Y3[10][16]`` is used instead of P216 (VfW)

OPT_Enable_b64a
---------------
::

    global OPT_Enable_b64a = true ## default false   AVS+ 

Use ``b64a`` instead of ``BRA[64]`` (VfW) 

OPT_Enable_PlanarToPackedRGB
----------------------------
::

    global OPT_Enable_PlanarToPackedRGB = true ## default false   AVS+ 

Convert Planar RGB to packed RGB (VfW) 

*   Planar RGB 8, 10, 12, 14 and 16 bits are reported as ``G3[0][8]``, ``G3[0][10]``, ``G3[0][12]``, ``G3[0][14]`` and ``G3[0][16]`` fourCC codes 
*   Planar RGBA 8, 10, 12, 14 and 16 bits are reported as ``G4[0][8]``, ``G4[0][10]``, ``G4[0][12]``, ``G4[0][14]`` and ``G4[0][16]`` 

When these FourCC codes are not handled through VfW, use ``OPT_Enable_PlanarToPackedRGB=true``. 

Avisynth+ will convert the clip from planar to RGB64 (packed 16bit RGB) and will negotiate this format instead


Changelog
~~~~~~~~~
+----------------+------------------------------------------------------------+
| Version        | Changes                                                    |
+================+============================================================+
| Avisynth 3.6.1 | | Added "SetCacheMode" (Neo addition)                      |
|                | | Added "SetMemoryMax" type and index options              |
+----------------+------------------------------------------------------------+
| Avisynth 3.6.0 | Added "SetMaxCPU"                                          |
+----------------+------------------------------------------------------------+

--------

Back to :doc:`Internal functions <syntax_internal_functions>`.


$Date: 2025-02-25 15:25:59-05:00 $

.. _planar: http://avisynth.nl/index.php/Planar
.. _memory alignment used in the AVIFile output emulation (not yet written):
    http://avisynth.nl/index.php/AVIFile_output_emulation
