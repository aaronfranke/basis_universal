# basis_universal_uastc_hdr
Basis Universal Supercompressed LDR/HDR GPU Texture Transcoding System 

[![Build status](https://ci.appveyor.com/api/projects/status/87eb0o96pjho4sh0?svg=true)](https://ci.appveyor.com/project/BinomialLLC/basis-universal)

----

Intro
-----

Basis Universal is an open source [supercompressed](http://gamma.cs.unc.edu/GST/gst.pdf) LDR/HDR GPU texture interchange system from Binomial LLC that supports two intermediate file formats: the [.KTX2 open standard from the Khronos Group](https://github.khronos.org/KTX-Specification/), and our own ".basis" file format. These file formats support rapid transcoding to virtually any [GPU texture format](https://en.wikipedia.org/wiki/Texture_compression) released in the past ~25 years. Our overall goal is to simplify the encoding and efficient distribution of LDR and HDR GPU texture, image, and texture video content in a way that works well on any GPU.

The current system supports three modes: ETC1S, UASTC LDR, and UASTC HDR.

Links
-----

- [Release Notes](https://github.com/BinomialLLC/basis_universal/wiki/Release-Notes)

- [Online WebGL Examples](https://subquantumtech.com/uastchdr2/) 

- [UASTC HDR Example Images](https://github.com/BinomialLLC/basis_universal/wiki/UASTC-HDR-Examples)

Supported LDR GPU Texture Formats
---------------------------------

ETC1S and UASTC LDR files can be transcoded to:

- ASTC LDR 4x4 L/LA/RGB/RGBA 8bpp
- BC1-5 RGB/RGBA/X/XY
- BC7 RGB/RGBA
- ETC1 RGB, ETC2 RGBA, and ETC2 EAC R11/RG11
- PVRTC1 4bpp RGB/RGBA and PVRTC2 RGB/RGBA
- ATC RGB/RGBA and FXT1 RGB
- Uncompressed LDR raster image formats: 8888/565/4444

Supported HDR GPU Texture Formats
---------------------------------

UASTC HDR files can be transcoded to:
- ASTC HDR 4x4 RGB 8bpp
- BC6H RGB
- Uncompressed HDR raster image formats: RGB_16F/RGBA_16F (half float/FP16 RGB, 48 or 64bpp), 32-bit shared exponent [RGB_9E5](https://registry.khronos.org/OpenGL/extensions/EXT/EXT_texture_shared_exponent.txt)

Supported Texture Compression Modes
-----------------------------------

1. [ETC1S](https://github.com/BinomialLLC/basis_universal/wiki/.basis-File-Format-and-ETC1S-Texture-Video-Specification): A roughly .3-3bpp low to medium quality supercompressed mode based off a subset of ETC1 called "ETC1S". This mode supports variable quality vs. file size levels (like JPEG), alpha channels, built-in compression, and texture arrays optionally compressed as a video sequence using skip blocks ([Conditional Replenishment](https://en.wikipedia.org/wiki/MPEG-1)). This mode can be rapidly transcoded to all of the supported LDR texture formats.

2. [UASTC LDR](https://richg42.blogspot.com/2020/01/uastc-block-format-encoding.html): An 8 bits/pixel LDR high quality mode. UASTC LDR is a 19 mode subset of the standard [ASTC LDR](https://en.wikipedia.org/wiki/Adaptive_scalable_texture_compression) 4x4 (8bpp) texture format, but with a custom block format containing transcoding hints. Transcoding UASTC LDR to ASTC LDR and BC7 are particularly fast and simple, because UASTC LDR is a common subset of both BC7 and ASTC. The transcoders for the other texture formats are accelerated by several format-specific hint bits present in each UASTC LDR block.

This mode supports an optional [Rate-Distortion Optimizated (RDO)](https://en.wikipedia.org/wiki/Rate%E2%80%93distortion_optimization) post-process stage that conditions the encoded UASTC LDR texture data in the .KTX2/.basis file so it can be more effectively LZ compressed. More details [here](https://github.com/BinomialLLC/basis_universal/wiki/UASTC-implementation-details).

Here is the [UASTC LDR specification document](https://github.com/BinomialLLC/basis_universal/wiki/UASTC-Texture-Specification).

3. [UASTC HDR](https://github.com/BinomialLLC/basis_universal/wiki/UASTC-HDR-Texture-Specification-v1.0): An 8 bits/pixel HDR high quality mode. This is a 24 mode subset of the standard [ASTC HDR](https://en.wikipedia.org/wiki/Adaptive_scalable_texture_compression) 4x4 (8bpp) texture format. It's designed to be high quality, supporting the 27 partition patterns in common between BC6H and ASTC, and fast to transcode with very little loss (typically a fraction of a dB PSNR) to the BC6H HDR texture format. Notably, **UASTC HDR data is 100% standard ASTC texture data**, so no transcoding at all is required on devices or API's supporting ASTC HDR. This mode can also be transcoded to various 32-64bpp uncompressed HDR texture/image formats.

Here is the [UASTC HDR specification document](https://github.com/BinomialLLC/basis_universal/wiki/UASTC-HDR-Texture-Specification-v1.0), and compressed [example images](https://github.com/BinomialLLC/basis_universal/wiki/UASTC-HDR-Examples).

### Other Features

Both .basis and .KTX2 files support mipmap levels, texture arrays, cubemaps, cubemap arrays, and texture video, in all three modes. Additionally, .basis files support non-uniform texture arrays, where each image in the file can have a different resolution or number of mipmap levels.

In ETC1S mode, the compressor is able to exploit color and pattern correlations across all the images in the entire file using global endpoint/selector codebooks, so multiple images with mipmaps can be stored efficiently in a single file. The ETC1S mode also supports short video sequences, with skip blocks (Conditional Replenishment) used to not send blocks which haven't changed relative to the previous frame.

The LDR image formats supported for reading are .PNG, [.DDS with mipmaps](https://learn.microsoft.com/en-us/windows/win32/direct3ddds/dx-graphics-dds-pguide), .TGA, .QOI, and .JPG. The HDR image formats supported for reading are .EXR, .HDR, and .DDS with mipmaps. It can write .basis, .KTX2, .DDS, .KTX (v1), .ASTC, .OUT, .EXR, and .PNG files.

The system now supports loading basic 2D .DDS files with optional mipmaps, but the .DDS file must be in one of the supported uncompressed formats: 24bpp RGB, 32bpp RGBA/BGRA, half-float RGBA, or float RGBA. Using .DDS files allows the user to control exactly how the mipmaps are generated before compression.

Building
--------

The encoding library and command line tool have no required 3rd party dependencies that are not already in the repo itself. The transcoder is a single .cpp source file (in `transcoder/basisu_transcoder.cpp`) which has no 3rd party dependencies.

We build and test under:
- Windows x86/x64 using Visual Studio 2019/2022, MSVC, or clang
- Mac OSX (M1) with clang v15.0
- Ubuntu Linux with gcc v11.4 or clang v14
- Arch Linux ARM, on a [Pinebook Pro](https://pine64.org/devices/pinebook_pro/), with gcc v12.1.

Under Windows with Visual Studio you can use the included `basisu.sln` file. Alternatively, you can use cmake to create new VS solution/project files.

To build, first [install cmake](https://cmake.org/), then:

```
cd build
cmake ..
make
```

To build with SSE 4.1 support on x86/x64 systems (encoding is roughly 15-30% faster), add `-DSSE=TRUE` to the cmake command line. Add `-DOPENCL=TRUE` to build with (optional) OpenCL support. Use `-DCMAKE_BUILD_TYPE=Debug` to build in debug. To build 32-bit executables, add `-DBUILD_X64=FALSE`.

After building, the native command line tool used to create, validate, and transcode/unpack .basis/.KTX2 files is `bin/basisu`.

### Testing the Codec

The command line tool includes some automated LDR/HDR encoding/transcoding tests:

```
cd ../bin
basisu -test
basisu -test_hdr
```

To test the codec in OpenCL mode (must have OpenCL libs/headers/drivers installed and have compiled OpenCL support in by running cmake with `-DOPENCL=TRUE`):

```
basisu -test -opencl
```

Compressing and Unpacking .KTX2/.basis Files
--------------------------------------------

- To compress an LDR sRGB PNG/QOI/TGA/JPEG/DDS image to an ETC1S .KTX2 file, at quality level 255 (the highest):

`basisu -q 255 x.png`

- For a linear LDR image, in ETC1S mode, at default quality (128):

`basisu -linear x.png`

- To compress to UASTC LDR, which is much higher quality than ETC1S:

`basisu -uastc x.png`

- To compress an [.EXR](https://en.wikipedia.org/wiki/OpenEXR), [Radiance .HDR](https://paulbourke.net/dataformats/pic/), or .DDS HDR image to a UASTC HDR .KTX2 file:

`basisu x.exr`

Alternatively, LDR images (such as .PNG) can be compressed to UASTC HDR by specifying `-hdr`. By default, LDR images, when compressed to UASTC HDR, are first converted from sRGB to linear light before compression. This conversion step can be disabled by specifying `-hdr_ldr_no_srgb_to_linear`. 

Importantly, for best quality, you should **supply basisu with original uncompressed source images**. Any other type of lossy compression applied before basisu (including ETC1/BC1-5, BC7, JPEG, etc.) will cause multi-generational artifacts to appear in the final output textures. 

### Some Useful Command Line Options

- `-fastest` (which is equivalent to `-uastc_level 0`) puts the UASTC LDR/HDR encoders in their fastest (but lower quality) modes. 

- `-slower` puts the UASTC LDR/HDR encoders in higher quality but slower modes (equivalent to `-uastc_level 3`). The default level is 1, and the highest is 4 (which is quite slow).

- `-q X`, where X ranges from [1,255], controls the ETC1S mode's quality vs. file size tradeoff level. 255 is the highest quality, and the default is 128.

- `-debug` causes the encoder to print internal and developer-oriented verbose debug information.

- `-stats` to see various quality (PSNR) statistics. 

- `-linear`: ETC1S defaults to sRGB colorspace metrics, UASTC LDR currently always uses linear metrics, and UASTC HDR defaults to weighted RGB metrics (with 2,3,1 weights). If the input is a normal map, or some other type of non-sRGB (non-photographic) texture content, be sure to use `-linear` to avoid extra unnecessary artifacts. (Angular normal map metrics for UASTC LDR/HDR are definitely doable and on our TODO list.)

- Specifying `-opencl` enables OpenCL mode, which currently only accelerates ETC1S encoding.

- The compressor is multithreaded by default, which can be disabled using the `-no_multithreading` command line option. The transcoder is currently single threaded, although it is thread safe (i.e. it supports decompressing multiple texture slices in parallel).

More Example Command Lines
--------------------------

- To compress an sRGB PNG/QOI/TGA/JPEG/DDS image to an RDO (Rate-Distortion Optimization) UASTC LDR .KTX2 file with mipmaps:

`basisu -uastc -uastc_rdo_l 1.0 -mipmap x.png`

`-uastc_rdo_l X` controls the RDO ([Rate-Distortion Optimization](https://en.wikipedia.org/wiki/Rate%E2%80%93distortion_optimization)) quality setting. The lower this value, the higher the quality, but the larger the compressed file size. Good values to try are between .2-3.0. The default is 1.0.

- To add automatically generated mipmaps to a ETC1S .KTX2 file, at a higher than default quality level (which ranges from [1,255]):

`basisu -mipmap -q 200 x.png`

There are several mipmap options to change the filter kernel, the filter colorspace for the RGB channels (linear vs. sRGB), the smallest mipmap dimension, etc. The tool also supports generating cubemap files, 2D/cubemap texture arrays, etc. To bypass the automatic mipmap generator, you can create LDR or HDR uncompressed [.DDS texture files](https://learn.microsoft.com/en-us/windows/win32/direct3ddds/dx-graphics-dds-pguide) and feed them to the compressor.

- To create a slightly higher quality ETC1S .KTX2 file (one with higher quality endpoint/selector codebooks) at the default quality level (128) - note this is much slower to encode:

`basisu -comp_level 2 x.png`

On some rare images (ones with blue sky gradients come to bind), you may need to increase the ETC1S `-comp_level` setting, which ranges from 1,6. This controls the amount of overall effort the encoder uses to optimize the ETC1S codebooks and the compressed data stream. Higher comp_level's are *significantly* slower. 

- To manually set the ETC1S codebook sizes (instead of using -q), with a higher codebook generation level (this is useful with texture video):

`basisu x.png -comp_level 2 -max_endpoints 16128 -max_selectors 16128`

- To [tonemap](https://en.wikipedia.org/wiki/Tone_mapping) an HDR .EXR or .HDR image file to multiple LDR .PNG files at different exposures, using the Reinhard tonemap operator:

`basisu -tonemap x.exr`

- To compare two LDR images and print PSNR statistics:

`basisu -compare a.png b.png`

- To compare two HDR .EXR/.HDR images and print FP16 PSNR statistics:

`basisu -compare_hdr a.exr b.exr`

See the help text for a complete listing of the tool's command line options. The command line tool is just a thin wrapper on top of the encoder library.

Unpacking .KTX2/.basis files to .PNG/.EXR/.KTX/.DDS files
---------------------------------------------------------

You can either use the command line tool or [call the transcoder directly](https://github.com/BinomialLLC/basis_universal/wiki/How-to-Use-and-Configure-the-Transcoder) from JavaScript or C/C++ code to decompress .KTX2/.basis files to GPU texture data or uncompressed image data. To unpack a .KTX2 or.basis file to multiple .png/.exr/.ktx/.dds files:

`basisu x.ktx2`

Use the `-no_ktx` and `-etc1_only`/`-format_only` options to unpack to less files. 

`-info` and `-validate` will just display file information and not output any files. 

The written mipmapped, cubemap, or texture array .KTX/.DDS files will be in a wide variety of compressed GPU texture formats (PVRTC1 4bpp, ETC1-2, BC1-5, BC7, etc.), and to our knowledge there is unfortunately (as of 2024) still no single .KTX or .DDS viewer tool that correctly and reliably supports every GPU texture format that we support. BC1-5 and BC7 files are viewable using AMD's Compressonator, ETC1/2 using Mali's Texture Compression Tool, and PVRTC1 using Imagination Tech's PVRTexTool. [RenderDoc](https://renderdoc.org/) has a useful texture file viewer for many formats. The Mac OSX Finder supports previewing .EXR and .KTX files in various GPU formats. The Windows 11 Explorer can preview .DDS files. The [online OpenHDR Viewer](https://viewer.openhdr.org/) is useful for viewing .EXR/.HDR image files. 

WebGL Examples
--------------

The "WebGL" directory contains three simple WebGL demos that use the transcoder and compressor compiled to [WASM](https://webassembly.org/) with [emscripten](https://emscripten.org/). See more details [here](webgl/README.md).

![Screenshot of 'texture' example running in a browser.](webgl/texture_test/preview.png)
![Screenshot of 'gltf' example running in a browser.](webgl/gltf/preview.png)
![Screenshot of 'encode_test' example running in a browser.](webgl/ktx2_encode_test/preview.png)

Building the WASM Modules with [Emscripten](https://emscripten.org/) 
--------------------------------------------------------------------

Both the transcoder and encoder may be compiled using emscripten to WebAssembly and used on the web. A set of JavaScript wrappers to the codec, written in C++ with emscripten extensions, is located in `webgl/transcoding/basis_wrappers.cpp`. The JavaScript wrapper supports nearly all features and modes, including texture video. See the README.md and CMakeLists.txt files in `webgl/transcoder` and `webgl/encoder`. 

To build the WASM transcoder, after installing emscripten:

```
cd webgl/transcoder/build
emcmake cmake ..
make
```

To build the WASM encoder:

```
cd webgl/encoder/build
emcmake cmake ..
make
```

There are two simple encoding/transcoding web demos, located in `webgl/ktx2_encode_test` and `webgl/texture_test`, that show how to use the encoder's and transcoder's Javascript wrapper API's.

Low-level C++ Encoder/Transcoder API Examples
---------------------------------------------

Some simple examples showing how to directly call the C++ encoder and transcoder library API's are in `example/examples.cpp`.

ETC1S Texture Video Tips
------------------------

ETC1S texture video support was a stretch goal of ours. Videos are significantly more challenging than textures, and supporting them helped us create a better looking system overall, as well as helping us gain experience with video. The current system only supports I-frames and P-frames with skip blocks, however it does use global endpoint/selector codebooks across all frames in the texture video sequence. Currently, the first frame is always an I-frame, and all subsequent frames are P-frames, although this current limitation is not imposed by the file format itself, just the API.

Mipmapping and alpha channels are also supported in ETC1S texture video mode. Internally, texture video files are treated as 2D texture arrays with an extra layer of compression: skip blocks on P-frames, and I-frames with no skip blocks. The global selector/endpoint codebooks are applied to all video frames.

Texture video stresses the encoder beyond its typical use, so some extra configuration is typically necessary. For nearly maximum possible achievable ETC1S mode quality with the current format and encoder (completely ignoring encoding speed!), use:

`-comp_level 5 -max_endpoints 16128 -max_selectors 16128 -no_selector_rdo -no_endpoint_rdo`

Level 5 is extremely slow, so unless you have a very powerful machine, levels 1-4 are recommended. "-no_selector_rdo -no_endpoint_rdo" are optional. Using them hurts rate-distortion performance, but they increase quality. An alternative is to use -selector_rdo_thresh X and -endpoint_rdo_thresh, with X ranging from [1,2] (higher=lower quality/better compression - see the tool's help text).

To compress small video sequences, using tools like ffmpeg and VirtualDub, first uncompress the video frames to multiple individual .PNG files:

`ffmpeg -i input.mp4 pic%04d.png`

Then, to compress the first 200 frames to a .basis file (.KTX2 works too):

`basisu -basis -comp_level 2 -tex_type video -multifile_printf "pic%04u.png" -multifile_num 200 -multifile_first 1 -max_selectors 16128 -max_endpoints 16128 -endpoint_rdo_thresh 1.05 -selector_rdo_thresh 1.05`

For ETC1S video encoding, the more cores and memory your machine has, the better. BasisU is intended for smaller videos of a few dozen seconds or so. On a powerful enough machine you should be able to encode up to a few thousand 720P frames using a single set of codebooks. The `webgl_videotest` directory contains a very simple (in progress) video viewer.

For texture video, use `-comp_level 2` or 3. The default is 1, which isn't quite good enough for texture video. Higher comp_level's result in reduced ETC1S artifacts.

The .basis file will contain multiple ETC1S image frames (or slices) in a large 2D texture array, all using the same global codebooks, which you can retrieve using the transcoder's image API. The system now supports [conditional replenishment](https://en.wikipedia.org/wiki/MPEG-1) (CR, or "skip blocks"). CR can reduce the bitrate of some videos (highly dependent on how dynamic the content is) by over 50%. In texture video mode, the images must be requested from the transcoder in sequence from first to last, and random access is only allowed to I-Frames.

Be sure to experiment with increasing the endpoint RDO threshold (-endpoint_rdo_thresh X). This setting controls how aggressively the compressor's backend will combine together nearby blocks so they use the same block endpoint codebook vectors, for better coding efficiency. X defaults to a modest 1.5, which means the backend is allowed to increase the overall color distance by 1.5x while searching for merge candidates. The higher this setting, the better the compression, with the tradeoff of more block artifacts. Settings up to ~2.25 can work well, and make the codec stronger. "-endpoint_rdo_thresh 1.75" is a good setting on many textures.

For video, `-comp_level 1` should result in decent results on most clips. For less banding, level 2 can make a big difference. This is still an active area of development, and quality/encoding perf. will improve over time.

For more info on controlling the ETC1S encoder's quality vs. encoding speed tradeoff, see [ETC1S Compression Effort Levels](https://github.com/BinomialLLC/basis_universal/wiki/ETC1S-Compression-Effort-Levels).

Installation using the vcpkg dependency manager
-----------------------------------------------

You can download and install Basis Universal using the [vcpkg](https://github.com/Microsoft/vcpkg/) dependency manager:

    git clone https://github.com/Microsoft/vcpkg.git
    cd vcpkg
    ./bootstrap-vcpkg.sh
    ./vcpkg integrate install
    vcpkg install basisu

The Basis Universal port in vcpkg is kept up to date by Microsoft team members and community contributors. If the version is out of date, please [create an issue or pull request](https://github.com/Microsoft/vcpkg) on the vcpkg repository.

Repository Licensing with REUSE
-------------------------------

The repository has been updated to be compliant with the REUSE license
checking tool (https://reuse.software/). See the `.reuse` subdirectory.

External Tool Links
-------------------

[Online .EXR HDR Image File Viewer](https://viewer.openhdr.org/)

[Windows HDR + WCG Image Viewer](https://13thsymphony.github.io/hdrimageviewer/) - A true HDR image viewer for Windows. Also see [the github repo](https://github.com/13thsymphony/HDRImageViewer).

[RenderDoc](https://renderdoc.org/)

[AMD Compressonator](https://gpuopen.com/gaming-product/compressonator/)

[Microsoft's DirectXTex](https://github.com/microsoft/DirectXTex)

[PVRTexTool](https://www.imgtec.com/developers/powervr-sdk-tools/pvrtextool/)

[Mali Texture Compression Tool](https://community.arm.com/support-forums/f/graphics-gaming-and-vr-forum/52390/announcement-mali-texture-compression-tool-end-of-life) - Now deprecated

For more useful links, papers, and tools/libraries, see the end of the [UASTC HDR texture specification](https://github.com/BinomialLLC/basis_universal/wiki/UASTC-HDR-Texture-Specification-v1.0).

----

E-mail: info @ binomial dot info, or contact us on [Twitter](https://twitter.com/_binomial)

Here's the [Sponsors](https://github.com/BinomialLLC/basis_universal/wiki/Sponsors-and-Supporters) wiki page.
