
a8rawconv

Copyright © 2014-2023 Avery Lee
Introduction

a8rawconv is a utility for converting between raw and decoded disk images, taking advantage of recent availability of high-quality devices for reading and writing raw flux-level images of floppy disks. This allows for easy and fast methods of creating usable images of floppy disks without a modified disk drive, particularly unusually encoded or even copy protected floppy disks. Decoded disk images are better suited for testing in an emulator or on real hardware using a disk simulator, and provide a way to verify whether a disk image is valid before archiving.
Basic usage
Running the program (Windows)

The converter is provided in binary form as a Win32 command line utility called a8rawconv.exe, which can be run on Windows 7 or later, on either a 32-bit or 64-bit system. To use it, open a Command Prompt and run the program from the command line. This allows command-line arguments to be specified and the output to be seen.

Source code is also provided so that the converter can be rebuilt or modified. Visual Studio 2022 or newer is required.
Running the program (Linux or macOS)

Although binaries are not provided for other platforms, the source code is included so that it can be compiled. The core is designed to be portable to most systems that have at least a C++11 compiler, with the serial device functionality also available for Linux systems.

To compile the converter, build compileall.cpp with a C++ compiler in C++11 or C++14 mode. This file is a bulk file that includes all the other files, avoiding the need for a build system. This is the same build process used for the Release build on Windows. Typically, this will suffice (replace with your preferred C++ build system as appropriate):

    g++ -O2 -std=c++14 compileall.cpp 

Command-line syntax

a8rawconv is a command-line program that is invoked with the following syntax:

    a8rawconv [options] source-image destination-image 

Basic operations just take two filenames, the path to the source file and the path to the destination file. The specific operation, such as decoding, encoding, or format conversion, is determined by the filename extensions.

Many options are also supported to tweak the operation of the program. The list of options is documented within the program itself, which can be seen simply by running the program with no arguments:

    a8rawconv 

Decoding raw disk images

The main use of a8rawconv is to convert from a raw disk image produced by imaging hardware to a more usable decoded disk image. This can be done by running a8rawconv with the input and output names:

    a8rawconv track00.0.raw mydisk.atx
    a8rawconv mydisk.scp mydisk.atx 

Raw images can be produced from physical floppy disks either by KryoFlux or SuperCard Pro imaging hardware. By default, the input format, decoding parameters and output format are determined by the extension on the input and output filenames; .atx selects the VAPI/ATX format with Atari 8-bit decoding parameters. Similarly, .nib selects the NIB disk format with Apple II 5.25" decoding parameters. These can be overridden with the -if, -of, and -d parameters.

When using KryoFlux raw streams (track00.0.raw, track02.0.raw, etc.), a 96/135 tracks per inch (TPI) drive and an 70/80 track dump are assumed by default. Thus, the converter will dump even tracks. When using a 48 TPI drive and 35/40 track dumps, the -tpi switch must be used to change the interpretation to 48 TPI.

Side 2 of a KryoFlux double-sided raw stream set can be decoded as a single-sided disk by specifying track00.1.raw as the starting filename. (This will only work if the disk drive has been adjusted, since normally the top and bottom heads are offset.)
Interoperability problem with SuperCard Pro images

The SuperCard Pro (.scp) format has some issues with ambiguity in the way that tracks are stored in the file, which can make it difficult to reliably determine whether the tracks are stored with physical 48tpi or 96tpi spacing. This can cause interoperability problems between programs where one program cannot read .scp images written by another program. Current versions of a8rawconv use heuristics to try to interpret the file, but incompatibilities may still occur, particularly depending on the value of the disk type ID in the SCP image header.

To combat this problem, the -if and -of switches can be used to force a particular interpretation of the track list. All four combinations of single-sided, double-sided, 40-track (48tpi), and 80-track (96tpi) are supported. For instance, if the other program is expecting the track list to be emitted with two sides and 40-track spacing, the scp-ds40 format ID will cause a8rawconv to read and write the .scp file with this layout.

For Atari disks specifically, a8rawconv follows the actual behavior of the official SuperCard Pro software up through version 2.20, which is to write images with the Atari 400/800 disk type ID with a track layout of 40 tracks of two sides each in the first 80 entries of the track list. This means that for a single sided image, only every other track entry is populated or used. This is regardless of whether the image header specifies 48tpi or 96tpi, as that flag indicates the properties of the drive used and not the image layout.
Imaging physical floppy disks

Typically, the first step is to create a raw image of a floppy disk using the software that comes with the imaging hardware. There are a few things that should be kept in mind when doing this:

    Use the preservation modes of the imaging software when possible. This typically records five revolutions of all tracks on the disk. For older disk formats this often over-images the disk, but that's much better than finding out later you're missing part of it. Another reason to do this is that if the physical disk is deteriorating, you only want to do one pass on the disk because you may not get another chance. The safest approach is simply to grab a full five revolutions of all 80 tracks on both sides and sort it out later.
    For Atari 8-bit disks, you must have at least two revolutions imaged per track, and you must image 40 tracks. An index-aligned, one-rev image will not work because sectors can cross the index mark. A single revolution in splice mode also will not work.
    Atari 8-bit disks have 40 tracks. Recording with a straight C64 or Apple II profile won't work, because you'll only have 35 out of the 40 tracks.
    More revolutions are better because a8rawconv can use the duplicate data to salvage marginal sectors or distinguish stable from weak sectors.
    Currently, a8rawconv expects 80 track images when reading KryoFlux raw streams by default. 40-track (48 TPI) image sets can be used by supplying the -tpi 48 switch when decoding.
    While Atari drives spin at 288 RPM, and the PC drives used for imaging spin at either 300 RPM or 360 RPM, either speed is fine for imaging. The converter detects the speed of the imaging drive from the raw image and adjusts the decoding parameters for the difference in speed.

a8rawconv supports both KryoFlux raw streams (*.raw) and SuperCard Pro images (*.scp). Although there are differences between both the KryoFlux and SuperCard Pro hardware and software, both produce images suitable for conversion and neither has a significant advantage over the other in image quality or read reliability.

When archiving, it is always recommended that you keep the original raw images of a disk for archival purposes even if the converter successfully creates a fully functional decoded disk. This is because the decoded disk does not contain all of the information of the original disk. If it turns out that something is missing or the converter is updated, the raw source can be used to reconvert without using the physical disk again. Raw images compress well using standard compression tools.

In non-archival cases, this can be skipped. If you are decoding a disk containing a BASIC program you wrote as a kid, there won't be any copy protection on the disk and it's unlikely you'll care about the exact sector timing. Therefore, once you verify that the decoded disk is good, you don't need to keep the raw image. The raw images are so small in modern terms, though, that you might consider still keeping them around.

Note: Some versions of the SuperCard Pro software have a buggy profile for Atari 8-bit disks and will attempt to create an image using a single index-aligned revolution. This will result in lost sectors on most disks due to sectors being split across the index mark and a8rawconv will issue a warning if it sees this in the image. The imaging parameters must be manually adjusted to at least two revolutions.
Reading directly from physical floppy disks

a8rawconv can also use a SuperCard Pro device to directly read from a floppy disk. The SuperCard Pro software must be installed first to install the USB communications driver, after which the converter can access the hardware using special filenames:

    (Windows):
        scp0:96tpi (read from first drive, using 96tpi track density)
        scp1:48tpi (read from second drive, using 48tpi track density)
        scp0:96tpi:com4 (read from first drive as 96tpi, overriding device path to COM4)
    (Linux):
        scp0:96tpi:/dev/ttyUSB0 (read from first drive, using 96 tpi track density, from device /dev/ttyUSB0)

Therefore, a physical disk can be imaged as follows:

    a8rawconv scp0:96tpi test.scp 

On Windows, a8rawconv will automatically search for the first COM port that is a USB connection to a SuperCard Pro device, specifically an FTDI-based USB serial port driver identified as VID_0403+PID_6015+SCP-JIMA. The Virtual COM Port (VCP) mode of the FTDI serial driver must be enabled for this to work. It can be enabled in Device Manager.

On Linux, the serial device name must be manually specified, and you may also need to run with elevated (superuser) privileges.

Reading directly from a floppy disk can be useful when doing testing or transferring data. It is not recommended for old floppy disks since a raw image is not created during the process, and repeating the decoding process with different settings requires re-reading the disk, which is slower and puts more stress on the disk. Archiving should always be done by creating a preservation-level raw image once and then re-decoding the raw image as necessary.
Imaging to flux

When imaging from a physical disk, the destination can either be a raw flux image (e.g. disk.scp) or a decoded image type (e.g. disk.atx). The best target depends on the disk. In most cases, it is best to image directly to a raw flux image first:

    a8rawconv scp0:96tpi disk.scp 

This will by default record a full five revolutions of each track in the image, in one pass on the disk. The image can be then be decoded multiple times from that image without requiring another pass on the original disk. More importantly, it can be re-decoded later again if something is discovered problematic with an earlier decode. Taking a preservation-level image of the disk in this manner is the safest way to deal with rare disks that may be starting to fail.

There are a couple of other switches that can be useful when doing a raw image. One is the -revs switch, which sets the revolution count when reading from an SCP device. The range is 1-5 revs, though 2 revs are minimum for non-index aligned disks like Atari disks. Reducing the rev count with -revs 2 is useful for faster imaging of a non-critical disk in good condition, and will also produce a smaller .scp image.

The other useful switch is -g, which overrides the disk geometry. a8rawconv will normally determine the best geometry based on the input and output formats, but for a raw imaging operation there is no such information. In this case, it will assume single-sided 40 tracks, which is enough to image up to Atari double-density but may be inadequate for other formats. -g allows the track and side count to be overridden, such as -g 80,2 for double-sided 80 tracks. This also allows overdumping up to 42 or 84 tracks for disks that have extra information outside of the normal track range. For truly blind cases where the disk format is totally unknown, it can be a good idea to just dump a full 83 tracks on both sides -- the unused tracks can just be ignored later if they turn out to not have any data.
Imaging to a decoded image

On the other hand, sometimes a disk isn't that precious and the two step process isn't convenient. For instance, if you just wrote out a disk from a 1050 and just want to image it, you can go directly from the physical disk to decoded image:

    a8rawconv scp0:96tpi disk.atx 

In this case, a8rawconv will both read and decode the disk in one pass. Of course, if something is wrong and the decoding parameters need to be adjusted, the disk will also need to be read again.
Assessing the quality of an image

The converter has a few features to assess the quality of a decoded image. The simplest is just how many sectors were extracted. For Atari 8-bit single density disks — the most common format — the converter should report no missing sectors or sectors with errors:

Writing ATX file: e:\test.atx
0 missing sectors, 0 phantom sectors, 0 sectors with errors

On the other hand, you might see something like this:

Writing ATX file: e:\test.atx
WARNING: Track 20, sector 16: Multiple sectors found at the same position
         0.81 but different bad data. Encoding weak sector at offset 25.
WARNING: Track 20: Missing sectors: 18.
1 missing sector, 0 phantom sectors, 1 sector with errors

If your disk is supposed to be just a normal DOS disk, this means you have some problems. If it's a commercial release, and especially if it is copy protected, then this might be expected. In the latter case, some experience with copy protection techniques is helpful and testing the decoded disk image is recommended.
Tweaking the decoding period

a8rawconv is designed to have good defaults so that most of the time conversion can be performed without a string of command-line switches. However, sometimes tweaking is necessary. One useful switch is the -p switch, which tweaks the default bit cell period used by the decoder, as a percentage of the normal value. This can be used to shrink or stretch the expected length of a bit slightly to nudge the decoder toward success when it is having trouble with a marginal disk:

    a8rawconv -p 98 disk.scp disk.atx 

The recommended approach is to try small adjustments in the range of 95-105 and watch the number of errors to zero in on the best value. Don't try crazy values like 70 or 130 to start with; you're likely to just get complete failure to decode the track. Watch out for when the decoder starts missing sectors entirely, as past that point the error count may drop because the decoder stops seeing some of the sectors.

The -t switch is also useful when tweaking decoding settings. It limits decoding to a single track on the disk, filtering out errors from tracks other than the one you're focusing on:

    a8rawconv -t 27 disk.scp disk.atx 

Note that this also causes only that track to be emitted into the output file, so this should not be used to produce the final disk image file. When reading from a SuperCard Pro, this will seek the mechanism to and read only the selected track.
Post-compensation

Floppy disk media is subject to a peak shift effect where flux transitions that are close together can have their read signals interact such that the pulses read farther apart than they were originally written. This effect can be serious enough to push flux transitions out of their original bit cell timing. For 4us FM and GCR formats, this is typically ignored, but for 2us MFM formats this is usually handled on the write side with write precompensation, where patterns known to cause this problem are pre-adjusted to combat the peak shift effect and read with approximately intended timing.

A notable exception to this is Macintosh 800K disks, which can show severe peak shift effects when read with a standard, constant-speed drive. This is partly due to the high flux density, with the 2us GCR format producing minimum transition spacing 35% tighter than a 2us MFM format. To combat this, a8rawconv by default applies post-compensation on the flux pattern when decoding to or analyzing for the Macintosh GCR encoding format. This moves the transitions back closer to each other so they are sufficiently within bitcell timing. Currently post-compensation is not used for any other encoding as they are assumed to either not need it or to have already had write precompensation applied.

If for some reason this default behavior is unsuitable, the post-compensation mode can be overridden with the -P switch.
Weak sectors

When multiple revolutions are available for a track, a8rawconv compares all copies of each physical sector to see whether they are the same. If they are not, it attempts to separate them into good and bad copies, and reports when this occurs:

WARNING: Track 38, sector  1: 1/3 bad sector reads discarded at position 0.25.

For a standard unprotected disk, this is a bad sign, as it means that the disk or either the imaging hardware is marginal, and could mean that either the disk is going bad or the imaging drive needs to be cleaned. Fortunately, the Cyclic Redundancy Check (CRC) stored with each sector allows the converter to tell if a sector is good, and this message means that the converter was able to recover the sector. This is one reason that five disk revolutions is recommended — the extra images of each sector increase the changes that the converter will be able to recover sectors from marginal disks.

When tweaking the decoder timing with -p, the ratio of good to bad sector reads can be used to aid in determining whether a change in timing is helping or hurting.

Note that in this case, "good" means that the sector was read correctly. Protected disks often have sectors that were written with deliberate CRC errors such that they are always read by the disk drive with errors; the converter can isolate good reads of these sectors in the same way, as both the CRC and data will be stable even if the CRC is incorrect.

In the event that the converter can't find at least two reads of the same sector that match, it will salvage as much common data as it can and then encode a weak sector:

WARNING: Track 38, sector  4: Multiple sectors found at the same position
         0.65 but different bad data. Encoding weak sector at offset 44.

For a regular disk, this is a bad sign as it means that the converter was not able to get a clean read of the sector. For protected disks, though, this can be valid as some disks were deliberately created with weak sectors that changed every time they were read by the disk drive. Typically there are only a small handful of these at most. When weak sectors are found, it is recommended that the decode be retried with different -p settings to try to determine whether these are truly weak sectors or bad reads of a stable sector.
Viewing track/sector layout

The -l switch causes the converter to display a crude representation of the sectors within each track. This is useful to identify unusual disk layouts:

 0 (18) |  6   8  10  12 14  16 18      1   3  5   7  9   11 13  15 17  2  4
 1 (18) |    8  10  12 14  16 18       1  3   5  7   9  11  13 15  17 2   4  6
 2 (18) |  8   10 12  14 16  18      1  3   5  7   9  11  13 15  17 2   4  6
 3 (18) | 8  10  12 14  16 18      1   3  5   7  9   11 13  15  17 2   4  6
 4 (18) |   10 12  14 16  18      1  3   5  7   9  11  13 15  17 2   4   6  8
 5 (18) | 10 12  14 16  18      1  3   5  7   9  11  13 15  17 2   4  6   8
 6 (18) |   12 14  16 18      1   3  5   7  9   11 13  15 17  2  4   6   8  10
 7 (18) | 12  14 16  18     1   3  5   7  9   11  13 15  17 2   4  6   8   10
 8 (18) |   14  16 18      1  3   5  7   9  11  13 15  17  2  4   6  8   10 12
 9 (18) | 14  16 18      1  3   5  7   9   11 13  15 17  2  4   6  8   10  12
10 (18) |   16 18      1   3  5   7  9   11 13  15  17 2   4  6   8  10  12 14
11 (18) | 16 18      1   3   5  7   9  11  13 15  17 2   4  6   8  10  12 14
12 (18) |  18       1  3   5  7   9  11  13 15  17 2   4  6   8  10  12 14  16

The left columns indicate the track number and number of physical sectors. The remainder of the line shows all sectors in the track, where the relative position of each sector number indicates the angular position of that sector. Remember that floppy disk tracks are a continuous circle, so the start and end of the line are the same position on the disk.

For a disk formatted on a standard Atari 810 or 1050 drive, even and odd sectors will be separated due to sector interleave, a reordering of the sectors to improve read/write performance. Disk drives with high-speed capability can format with a different sector interleave for better high-speed performance. Some commercial disks were written with a non-standard interleave and read slower than usual on a real drive.

The sectors on successive tracks will also be offset due to skew between the tracks. This is related to the delay when stepping between tracks during initial formatting. It is also possible for tracks to lack such skew and be aligned to the index mark:

 0 (18) |  18 7   14  3  10  17  6  13  2   9   16 5   12  1  8   15  4  11
 1 (18) |  18 7   14  3  10  17  6   13 2   9   16 5   12  1  8   15  4  11
 2 (18) |  18 7   14  3  10  17  6  13  2   9   16 5   12  1  8   15  4  11
 3 (18) |  18 7   14  3  10  17  6  13  2   9   16 5   12  1  8   15  4  11
 4 (18) |  18 7   14  3  10  17  6  13  2   9   16 5   12  1  8   15  4  11
 5 (18) |  18 7   14  3  10  17  6  13  2   9   16 5   12  1  8   15  4  11
 6 (18) |  18 7   14  3  10  17  6  13  2   9   16 5   12  1  8   15  4  11

Such a disk will read and write fine, but generally indicates that the disk was not created in an Atari drive. This example also shows a different sector interleave, which can cause the Atari to read the disk more slowly than usual.

Highly unusual patterns typically indicate copy protection, such as this track with 34 physical sectors:

 2 (34) | 2  34  56 7 8 9 10112 114 116 11  23  45  67  89 10111213115 117 1

Any track on a single-density Atari disk with more than 18 physical sectors on it has more than one duplicated or phantom sectors, where two or more physical sectors have the same sector number. This is used for protection schemes where the timing of the sector read determines which of the phantom sectors is read. Such a disk cannot be created on a stock Atari drive, although it can be created with a modified drive or with a hardware imager such as SuperCard Pro.

Finally, there is the possibility that the disk is just simply something else:

 0 ( 9) |   1      2      3      4      5      6      7       8      9
 1 ( 9) |   1      2      3      4      5      6      7       8      9
 2 ( 9) |   1      2      3      4      5      6      7       8      9
...
Writing ATX file: e:\test.atx
WARNING: Track  0, sector  1: Long sector of 512 bytes found.
WARNING: Track  0, sector  2: Long sector of 512 bytes found.
WARNING: Track  0, sector  3: Long sector of 512 bytes found.

For Atari disk formats, the converter attempts both FM and MFM decoding on each track, which means that it can pick up even non-Atari disk formats. In this case, the disk has 9 index-aligned sectors per track with 512 bytes per track instead of 18 sectors per track of 128 bytes each, meaning that it is likely a 360K MS-DOS disk, not an Atari disk.
Flippy disks

Some software shipped on "flippy" disks that could be flipped upside down to access another program variant or more data on the other side of the disk. Imaging these disks is problematic for a couple of reasons:

    Physically flipping the disk moves the index hole of the disk to the other side. The old floppy disk drives used with computers such as the Atari 800 and Apple II were often configured such that the index hole was ignored. PC drives, however, can require the index hole to read the disk, and may simply refuse to read a flipped disk.
    Track 0 on side 1 (bottom) is at the equivalent position for track -4 on side 2 (top). This means that the flip side cannot simply be imaged in a double-sided drive, as the first four tracks will be inaccessible.

It is possible to modify a floppy disk drive to circumvent one of these two problems. If the drive is modified to realign the second head, reading the disk image will produce track streams that are backwards. In that case, the -r option should be passed to a8rawconv so that it reads the track images backwards to match.
Multi-format disks

Ingenious fellows in the past found ways to create floppy disks that booted on more than one computer, by creatively interleaving sectors from the different formats on the same disk. a8rawconv does not directly support mixed-format disks, but can aid in detecting such disks and extracting out platform-specific versions. Unusual sector sizes or missing sectors/tracks in the layout map, particularly on tracks 1 or 18, are a possible sign of such a disk. In that case, conversion can be attempted using output formats for all platforms.

It is highly recommended that preservation-quality images of such multi-format disks be kept, as they are uncommon and currently can't be converted to a single decoded image.
Bit-level diagnosis

For experts dealing with troublesome disks, the -v option causes the converter to report additional information about the decoding process. There are four levels of verbosity:

    -v: Reports summary information about all physical sector images found in all revolutions.
    -vv: Reports more detailed information on each sector.
    -vvv: Reports bit-level decoding output:

    00 FF 00 FF 00 FF 00 FF 00 FF 00 FF 00 FF 00 FF | 15.64
    00 FF 00 FF 00 FF 00 FF 00 FF 00 FF 00 FF 00 FF | 15.71
    00 FF 00 FF 01 FF 03 FE 07 FC 0F F8 1F F1 3E E3 | 15.64
    7D C7 FB 8F F6 1F ED 3F DB 7F B7 FF 6F FF DF FF | 15.79
    BF FF 7F FF FF FF FF FF FF FF FF FF FF FF FF FF | 15.74
    FF FF FE FF FD FF FB FF F7 FF EF FF DF FF BF FF | 15.63
    7F FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF | 15.65
    FF FF FE FF FC FF F9 FF F2 FF E4 FF C8 FF 91 FF | 15.86

    Listed on the left is the complete dump of the shift register states, and on the right is the number of bit cell times measured for each group of 16 bits, composed of 8 clock bits and 8 data bits interleaved together.

    Because the relationship between clock and data bits and the byte alignment is not known until a sync byte is found, the bit-level dump simply shows all possible offsets. The layout of 16 shifts per line ensures that once the sync byte is found, successive bytes will be in a column. Above, highlighted in red, is the Data Address Mark (DAM) with $C7 clock pattern and $FB data byte, followed by the first four data bytes of the sector.

    Note that while byte alignment will always be consistent within a single address field or data field, there is no requirement for the address and data fields of a sector to share common byte alignment and the two will often be skewed by some bit offset from each other.
    -vvvv: Reports flux-level decoding output.

The higher verbosity levels can produce very large amounts of output, so restricting decoding to a single track with -t is highly recommended.
Flux analysis mode

Invoking a8rawconv with the -analyze switch selects flux analysis mode instead of regular conversion mode. In this mode, the flux timing of a raw disk is analyzed to show the distribution of flux timing and to give insight into either problems with the raw disk or if the wrong decoder is being used. -analyze takes one argument, the encoding mode to model to select appropriate bit cell timing. No output filename is needed with -analyze.

When -analyze is used, a histogram is produced for each track (-t is recommended to show one track at a time):

Track 0.0:
                       |   |
                       |   |
                       |   |
                       |   |
                       |   |
                       |   |
                       |   |
                       |   |
                       |   |
                       |   |
                       |   |
                       |   |
                       |   |
                       |   |
                       |   |
                       |   |   |
                       |   |   |
                       |   |   |
                       |   ||  |
                       |   ||  |
                       |   ||  |
                       |   ||  |        .
                       |   ||  |.       |||
                      ||   ||  ||       |||.
                      ||   ||  ||    |  ||||
                      ||   ||  ||    |. ||||
                      ||   ||  ||    || ||||
                      ||   ||  ||    ||||||| |                   |
                      ||.  ||  ||   .|||||||.|.|           .. ||||| |
                      ||| |||.||||  ||||||||||||          |||.|||||.|.|
-------------------------------------------------------------------------------------------
           .         2us         .          4us         .         6us         .

On a disk in good condition and without anomalies like bad sectors, the histogram peaks should be cleanly separated and centered around the expected timings for the encoding. In this case, this track uses Macintosh GCR encoding, which has a 2us bitcell and a maximum run of two consecutive 0 bits between 1 bits, so the expected timings are 2/4/6 us. As can be seen in the above, however, there is poor separation between 2us and 4us regions, which will make it hard for the decoder to distinguish 11 and 101 patterns.

The above histogram was generated with post-compensation disabled. With the default mac800k post-compensation mode enabled, the result is better:

Track 0.0:
                       |
                       |
                       |
                       |
                       |
                       | .
                       | |
                       | |
                       | |
                       | |
                       | ||
                       | ||
                       | ||
                       | ||
                       | ||                  .
                       | ||                 .|
                       ||||                 ||
                       ||||                 ||
                     | ||||                .||
                     | ||||                |||
                     | ||||                |||
                     ||||||                |||
                     ||||||.               |||.
                     |||||||               |||||
                     |||||||              .|||||                   |.
                    ||||||||              ||||||                   ||.
                   |||||||||              ||||||                  ||||
                   |||||||||              |||||||                 |||||
                   |||||||||             ||||||||                .|||||
                  ||||||||||.|           ||||||||               ||||||||
-------------------------------------------------------------------------------------------
           .         2us         .          4us         .         6us         .

The analyzer will work with any flux pattern regardless of the specified encoding, but the encoding is used to calibrate the time scale. For instance, using pc-360k will scale the time axis and use labels for a 2us bit cell at 300 RPM. The analysis encoding setting will also determine the default post-compensation mode unless overridden with the -P switch.
False successes

It is possible for the converter to believe it has successfully read a sector when it hasn't. The CRC16 used by the Atari disk format and other 177x compatible disk formats makes this unlikely. Unfortunately, 5.25" Apple II disks use a weak 8-bit XOR checksum algorithm that is more prone to false negatives and can fail to detect errors. This has been seen in practice with old disks that had many marginal sectors when read.

The only truly reliable way to verify a disk is to examine it manually, either by attempting to boot the decoded image or checking the files. The latter case can be annoying as the Apple II standard for text has the high bit set. The -I switch will invert bit 7 on decoded data so that text files can be more easily read in a hex editor. The resulting image will not work in an Apple II environment, however.
Converting between image formats
Downconverting to a simpler format

a8rawconv can also convert between decoded disk image formats. In this case, flux-level processing is skipped and the decoded sectors are converted directly. One use for this is to 'deprotect' an ATX file to ATR, when the disk image has no protection and does not need full timing data:

    a8rawconv test.atx test.atr 

This will only work if the ATX disk image is compatible with the ATR format, with sectors of uniform size and no unusual sector encodings. Timing information is dropped, any missing sectors are padded, and special sector information like CRC errors or weak bits are discarded. Warnings are displayed when this occurs to indicate that the image may not work, especially for copy protected images. However, this can also occur with unprotected, marginal disks that simply had bad sectors, and this may be fine if those sectors are not needed.
Upconverting to a more complex format

It is also possible to go the other way, by converting from ATR to ATX:

    a8rawconv test.atr test.atx 

In this case, a8rawconv will synthesize timing data for the disk image. This can be useful to enforce a specific sector interleave order, which can't be encoded in ATR but can in ATX. By default, the ATX image has sectors laid out with the sector interleave and track-to-track skew typical for an Atari 1050 disk drive. This can be overridden with the -i switch, as described in Overriding the default interleave.
Converting between flux formats

A KryoFlux (.raw) image can be converted to SuperCard Pro format (.scp):

    a8rawconv track00.0.raw image.scp 

For this conversion, partial revolutions at the beginning and end of each track image are dropped, and the flux timing samples are converted from KryoFlux 40ns precision to SuperCard Pro 25ns precision. No sector analysis is performed, however, and the flux samples are converted without interpreting them, so this is agnostic to disk format and will work with non-Atari formats.
Reincarnating an image

Finally, the most unusual but occasionally useful conversion mode is from a file format to the same file format:

    a8rawconv test.atx test2.atx 

In this case, a8rawconv will rewrite the disk image with the same contents but a new structure. This is useful if data is intact in the file but there is some issue with the structure that is causing compability problems with other software; if a8rawconv can read the file, it can re-write it with a clean structure. This includes fixing issues from older versions of a8rawconv.
Writing images to physical disks
Encoding and writing a pre-decoded image

With a SuperCard Pro device, a8rawconv can also write images back to physical disks. This is not typically useful for distribution as no one distributes software on physical floppy disks anymore, but can still be useful for testing or if floppy disk is the only way to get software onto a real computer.

To do so, simply specify the SuperCard Pro as the output device instead of the source:

    a8rawconv test.atr scp0:96tpi 

The converter will then encode the disk image to physical format and write them to the physical disk through the SCP. Pre-erasing or preformatting the disk is not necessary. Both Atari FM (single) and MFM (enhanced) formats can be encoded and written back to disk. Note that on an 80 track (96 TPI) drive, only the even tracks are written.

135tpi can also be specified as a synonym for 96tpi, as 3.5" drives are also 80 tracks but have 135 TPI track density.

It is also possible, though not recommended, to write out an .scp image instead:

    a8rawconv test.atx test.scp 

The reason this isn't recommended is that the .scp format always includes tracks from index-to-index and does not include information about appropriate splice points for the image. a8rawconv can deal with this because it has knowledge of the Atari disk format and will re-analyze the image and choose a splice point between sectors for ending the track write, but other tools may not and may tear a sector overlapping the index mark. In particular, using the stock SuperCard Pro software to write the image will likely fail and write a broken track unless the tracks are all index-aligned.

IMPORTANT: The image written back to disk will not be the same as the original, even though it will have the same sector data content. This is because the converter re-lays out the track according to the standard format expected by Atari disk drives, which can cause small timing differences from the original. If an accurate duplicate is required, it should be produced from the raw stream image using the duplication software supplied with the hardware.
Overriding the default interleave

Older computer systems could not read sectors at full speed from the disk drive and required the sectors to be interleaved for optimum ordering for the speed at which they could read the disk. Using the wrong interleave results in slow disk reads because the disk drive has to wait longer for each sector to arrive under the drive head when the computer requests it. On Atari disks, this typically results in sectors being read from the disk at about half the normal speed.

In a8rawconv, interleaving is usually handled automatically. When decoding a disk from a raw flux/nibble format or a format with timing information like ATX, the sector order and thus interleave pattern is already present. The case that matters is when converting from a format that doesn't have sector timing information, like ATR, to a raw format or a format with timing information. In that case, a8rawconv will apply a sector interleave appropriate for Atari disk geometry. The defaults are a 9:1 interleave for single density and double density, and a 13:1 interleave for enhanced density. This provides normal read speed on 1050 and XF551 disk drives with standard 19,200 baud communication. For 512 byte sectors, such as expected for PC and Amiga disks, a 1:1 interleave is applied.

In some cases, the default interleave may need to be overridden with the -i switch. This includes if a foreign disk format is being used that doesn't match the heuristics or if a disk needs to be re-interleaved for high-speed operation. For example, -i xf551-hs will cause a 9:1 interleave to be used for double-density disks for XF551 high-speed operation. Similarly, Indus GT CP/M disks need 1:1 interleave and should be written with -i none.

The -i option also affects track skew, the difference in timing between the start of one track and the next track. By default, a track skew of 8% of a rotation is applied unless 1:1 interleaving is being used, in which case no skew is applied and all tracks are index-aligned. For Atari disks without copy protection, track skew is not critical and only affects a minor delay when stepping between tracks -- all Atari-compatible disk drives must be able to read non-aligned tracks. Some non-Atari formats are naturally index aligned, however, and should not be encoded with track skew.
Problems during encoding

Plain disk images with standard geometry typically write out to a physical disk without problems. For protected disks, the converter has support for writing sectors with CRC errors, long sectors, phantom sectors, deleted sectors, and even weak sectors, and can write out usable versions of many protected disks. However, some types of encoding can still pose a problem for the encoder.

The main problem that can occur when writing out a protected disk is long tracks:

WARNING: Track 2, sectors 4 and 2 overlapped during encoding. Encoded track may not work.

This happens when there are phantom sectors on the track, and more sectors than can fit onto the track with standard encoding. When this happens, the -p option can be used to record bits onto the track at a faster rate, squeezing more data onto the track. This should be done with care as it reduces the read margin and can cause the disk to read unreliably or not at all. Also, as with decoding, small increments should be used when adjusting the period, around 95-105; starting with -p 50 will simply produce an unusable disk.

When writing long sectors with CRC errors, a8rawconv will write the sectors with their intended length but truncate the sectors at 128 bytes. This saves space on the track, but produces the same result as the standard disk interface only reports the first 128 bytes.

Occasionally, a disk image may have a track with an impossibly high number of sectors in it, such as 36 sectors per track, that can't be written out in usable form even with tight -p settings. This almost always means that the track was crafted to have overlapping sectors, where the data field of one sector overlaps the address field and even the data field of the next sector. This is possible because the floppy drive controller (FDC) does not validate the clock bits of data bytes. Unfortunately, decoded image formats do not store where sectors were overlapped, and a8rawconv does not currently have analysis algorithms to encode overlapping sectors.
Writing flux images

a8rawconv can also write raw flux images to disk, both from KryoFlux and SuperCard Pro images. This is done using the same syntax as for writing a decoded image:

    a8rawconv image.scp scp0:96tpi 

When writing a flux image, a8rawconv will first run a decoding pass as usual to identify and parse sectors from the image. However, those sectors are only used to identify the best points for starting each track write; the decoded sector is discarded and the original flux is then written to disk. This ensures that the splice point where the track write starts and where a few bits will be corrupted is safely between sectors where it will not affect reading the disk.

In order for this to work, the source image has to contain at least two full revolutions of flux for each track image, so that the analyzer can identify start and stop points that don't split a sector. This almost always requires a splice point other than at the index mark and thus in the middle of a revolution.

Only one revolution's worth of flux is extracted and written to disk; any additional redundant flux data is ignored. Thus, writing raw flux from a marginal image is not advisable as the selected range may have bad sector reads. This will go unnoticed when writing from flux as the sector decoder is skipped. For a marginal image, it is better to decode the flux to data first so that a8rawconv can check the sector CRCs and try to find the a good image for each sector, and then reassemble the good sector copies into a clean track.
Supported formats
Disk geometries

The main disk geometries supported by a8rawconv are listed below. In current versions, the disk geometry can also be varied with the -g switch, which is particularly useful when doing blind imaging of an unknown disk format. It takes two arguments, the track count and side count, so -g 80,2 uses an 80-track double-sided geometry instead.

Atari 8-bit single density (read/write)
    Single-sided, 40 tracks, 18 sectors per track, 128 bytes per sector, 288 RPM, FM encoding, 90K total. 
Atari 8-bit enhanced (medium) density (read/write)
    Single-sided, 40 tracks, 26 sectors per track, 128 bytes per sector, 288 RPM, MFM encoding, 130K total. 
Atari 8-bit double density (read/write)
    Single-sided, 40 tracks, 18 sectors per track, 256 bytes per sector, 288 RPM, MFM encoding, 180K total. 
Atari 8-bit DSDD (read/write)
    Double-sided, 40 tracks, 18 sectors per track, 256 bytes per sector, 300 RPM, MFM encoding, 360K total. This format is used by the Atari XF551 disk drive. 
Apple II 5.25" (read only)
    Single-sided, 35 tracks, 16 sectors per track, 256 bytes per sector, 300 RPM, 4µs GCR encoding, 160K total. 
Macintosh / Apple II 3.5"
    Single- or double-sided, 80 tracks, variable sectors per track, 512 bytes per sector, 394-590 RPM, 2µs GCR encoding, 400/800K total. 
PC
    Single- or double-sided, 40/80 track, 9-21 sectors per track, 512 bytes per sector, 300/360 RPM, MFM encoding, 160-1680K total. 
1771/1772 FDC compatible formats (diagnostic decoding only)
    360K MS-DOS, TRS-80, etc. formats are not supported for I/O, but can be decoded for diagnostic purposes due to all using the same basic format supported by the common floppy drive controller chips of the era. 

Disk image formats

ATR (read/write)
    Atari 8-bit decoded disk image. Widely supported by emulators, disk simulators, and disk image tools; supports all common disk geometries. This is the image format of choice for standard, unprotected Atari 8-bit disks and is the one that should be used by default. However, this disk format will not work for protected disks, as it has no place for timing, error, or duplicate sector metadata. 
ATX (read/write)
    Atari 8-bit extended decoded disk image. Known also by the name VAPI, the API originally published for software that established this format. Contains timing and error information and can encode most anomalies used in common copy protections. This format contains enough information to typically run the software, but only sometimes enough to recreate a working physical disk. Only 90K single density and 130K enhanced density disk geometries are supported. Software support is mediocre; all current emulators support it, but only some disk simulators and image tools do, and older tools or emulators do not. Notably, the older but still very popular Atari800WinPLus 4.0 emulator does not support ATX (there is a modified version, but it has some bugs). 
    Enhanced density ATX images are supported by a8rawconv but are not universally supported by other tools as it is a recent addition to the file format. 
XFD (read/write)
    Atari 8-bit decoded disk image, XFormer format. Widely supported by emulators, but supports only specific disk geometries (mainly 1050 compatible). This format is headerless and is simply a direct dump of all sectors in logical sector order. 
    In most cases, ATR is a better choice than XFD. There is one exception: XFD is the only common format that can store a full 256 bytes from boot sectors on a double density disk. This is important in rare cases, notably an Indus GT CP/M disk. 
DSK (write only)
    Macintosh or Apple II Unidisk 3.5" image, 400K or 800K. 
    Note: Support for Macintosh GCR decoding is preliminary, as the decoder currently has trouble with sectors on the outer zone of tracks. 
DO/DSK (read/write)
    Apple II decoded disk image, DOS 3.3 sector order. Widely supported by Apple II emulators and disk tools, but only stores unprotected disks in DOS 3.3-like format. 
PO (read/write)
    Apple II decoded disk image, ProDOS sector order. Widely supported by Apple II emulators and disk tools, but only stores unprotected disks in ProDOS-like format. 
NIB (read/write)
    Apple II "nibble" disk image. This is a disk image that stores partially decoded GCR data in the format used between the CPU and the Disk II controller. It is mostly a low-level format and can accommodate some protection encodings, but is a partially decoded format as it captures data after the Disk II controller has done byte syncing. In particular, it lacks timing information to distinguish between sync and data bytes. Supported by some emulators and disk tools. 
VFD/FLP (read/write)
    PC virtual floppy disk image. This is a dump of all decoded sectors in order with no headers or footers. Virtualization programs like VMWare and Virtual PC support this format, although they may not support all disk geometries. Typically 720K and 1.44MB are supported, but older formats not necessarily. 
    When converting a VFD image to flux, a8rawconv uses a 1:1 interleave by default as typically expected by PCs. 
    Some archivers, like 7-Zip, can read and extract from an MS-DOS filesystem in a VFD image. 
ADF (read/write)
    Amiga floppy disk image, 880K MFM encoded. This is a dump of all decoded sectors in order with no headers or footers. 
SCP (read/write)
    SuperCard Pro imaging format. Contains raw flux image from 1-5 rotations of tracks on disk in a single file, at 25ns sample resolution. The SCP format is suitable for both duplication and archival, if read/written with correct settings, and also a high-quality format for conversion to decoded formats. However, SCP images typically must be converted through a tool like a8rawconv to decoded formats like ATR/ATX for use with other tools or disk emulation hardware. 
RAW (read only)
    KryoFlux raw imaging format. Contains raw flux image from one or more rotations on the disk at 40ns sample resolution, in one file per track. The KryoFlux raw format is suitable for both duplication and archival, if read/written with correct settings, and also a high-quality format for conversion to decoded formats. However, KryoFlux raw images typically must be converted through a tool like a8rawconv to decoded formats like ATR/ATX for use with other tools or disk emulation hardware. 

DiskScript

A .diskscript file allows generation of specific flux patterns through a readable text file that can then be processed like any other raw flux image, typically for writing to a physical disk. It is designed to simplify writing patterns that are accepted by the FDC for regular sector read/write commands.

To use a DiskScript file, simply specify the .diskscript file as the source image instead of a regular image file. a8rawconv will then compile the script and either report any errors or run the script. The generated flux image is then treated as any other source for conversion or writing to physical disk.
Basic structure

DiskScript is modeled after the basic syntax of the C programming language, although with a much simpler structure. Statements are ended by a semicolon (;), and grouped as a block statement with curly braces ({}). Indentation is not significant and any amount of whitespace is the same as a single space. Comments may be single line with // or multi-line with /* and */. Hexadecimal constants are prefixed with 0x.

A disk script is generally structured as a series of tracks, with a list of commands to generate the flux transitions for each track. While creating a track, a cursor is advanced to track the current time or angular position within the track. Statements that add to the track advance the cursor by some time and possibly emit flux transitions within that time.

As DiskScript is oriented toward Atari disk formats, the standard emit statements are based around a 4us raw bit cell at 288 RPM, for an ideal capacity of about 3,255 data bytes per track. Flux can be written for other densities including MFM by using the low-level flux commands, but the higher-level commands like byte use Atari FM timing. The generated flux stream uses 25ns precision for timing, same as the SuperCard Pro.
Statements
byte value;

Emits flux transitions for an FM data byte with the specified value. The current position is assumed to be the center of the bit cell, so the first clock transition is written out after a delay of half a bit cell (2us), followed by the data transition another 4us later for a '1', or the next clock transition 8us later for a '0'. Eight pairs of clock and data cells are written and the cursor is advanced to the end of the last data bit cell (2us past last possible data transition for the last bit). Thus, the cursor is advanced by a total of 64us.
bytes value [, value...];

Writes one or more byte values from a comma-delimited list. This is just a shortcut for multiple bytes and is equivalent to emitting each byte out with a separate byte statement. The cursor is advanced by N*64us where N is the number of bytes in the list.
crc_begin;

Emits nothing, but marks the beginning of a region tracked for CRC-16 computation. Bytes emitted after this statement are included in the CRC up until the next crc_end statement, which ends the CRC computation and writes out the 16-bit CRC value.

When writing an FDC-compatible address or data field in FM, the CRC scope should include the DAM/IDAM and the address or data field payload. For MFM, the leading three $A1 sync bytes should also be included.
crc_end;

Writes the current CRC-16 value as two bytes, in big-endian order compatible with 177x/277x FDCs. The cursor is advanced by two byte times (128us).
flux delay;

Emits a flux transition after the specified delay. The delay is specified in FM bit cell units, so a value of 100 means a delay of 4us. Two flux 100; statements therefore write out the clock and data cells for a FM '1' bit.
no_flux delay;

Advances the cursor by the given percentage of FM bit cells without emitting any flux transitions. A value of 100 advances the cursor by 4us.
pad_bits count, bit-value;

Emits zero or more FM data bits with the given value, either 0 or 1 for a 0/1 bit. This emits both a 4us clock cell and a 4us data cell, with a clock transition after 2us and a possible data transition after 6us. The cursor is advanced by 8us.
repeat count child-statement;

Repeats a child statement the given number of times. The child statement may be a single statement or a block of statements grouped by curly braces:

repeat 10
    byte 0xFF;

repeat 10 {
    byte 0xFF;
    byte 0x00;
}

special_byte value;

Emits an FM byte with special clock transitions appropriate for an address or data mark, 0xC7. Eight pairs of clock and data cells are written with the first clock transition 2us after the starting position and the cursor is advanced by 64us. The byte value is usually either 0xFE for an ID address mark (IDAM) or 0xFB for a normal data address mark (DAM).
track track-number [, side-number] child-statement;

Creates a track with the given track number from 0-82, with the child statement creating the contents of the track. All statements that emit flux or advance the cursor must be within a track statement. The child statement is typically a block of statements:

track 0 {
    // write address field
    byte 0x00;
    crc_begin;
    special_byte 0xFE;
    bytes 0x02, 0x20, 0x09, 0x00;
    crc_end;

    repeat 15 {
            byte 0x00;
    }

    // write data field
    crc_begin;
    special_byte 0xFB;
    repeat 128 {
            byte 0xFF;
    }
    crc_end;
}

If side is specified, then it should be either 0 (bottom) or 1 (top). If omitted, side 0 is assumed.
geometry tracks, sides;

Sets the number of tracks per sides and sides in the disk geometry. For instance, geometry 40, 1; sets a 40-track, single-sided disk, while geometry 82, 2 sets an 82-track, double-sided disk. This must be done before setting up tracks.
Version history

a8rawconv 0.95

        KryoFlux image format
            KryoFlux streams with index marks at the beginning of the stream are now supported.
            Decoding side 2 of a KryoFlux stream set as a single sided disk is once again supported by specifying track00.1.raw as the base filename.
        SCP image format
            Fixed incorrect error when attempting to write a 48tpi image to a 48tpi SCP file.
        Decoding
            FM/MFM decoder no longer requires the first byte before a DAM to be $00. A 2793 FDC needs about four zero bits in FM to successfully read a sector, but not a full byte.
        Encoding
            Fixed encoding of MFM sectors that have no data field.
            Fixed incorrect header epilogue byte when encoding Apple II DOS 3.3 sectors.
            Encoder now has an -e precise option to use original sector positions and also maintains bit alignment through the written track.
            Write precompensation is now applied to the inner half of the disk when encoding MFM, to increase reliability of inner tracks.

a8rawconv 0.94

        General
            A precompiled x64 Windows build is now included.
            Disk format geometry can now be directly specified with the -g option. In particular, this allows dumping more than 40/80 tracks.
            Double-sided and 80-track formats are now supported.
            Added -H option to switch to 2us clock timing for 8" FM and 1us clock timing for high-density 3.5" MFM formats, and to enable high-density mode on disk drives connected to a SuperCard Pro device.
            .XFD files are now supported.
            .VFD/.FLP formats are now supported for PC disk formats.
            Relaxed MFM address field validation to ignore the side field and allow unusual sector size encodings accepted by 179X/279X FDCs.
            Added Macintosh 400K and 800K decoding support and DSK format.
            Added Amiga 880K decoding support and ADF format.
            Fixed handling of address CRC errors during disk encoding.
            Added -analyze to display a histogram of flux transition timing within each track.
            TPI-related options now accept 135 TPI as a synonym for 96 TPI. The TPI settings are used to control single-stepping or double-stepping on 80 track drives, but technically 96 TPI is only for 5.25" drives as 3.5" drives are 135 TPI.
        ATR image format
            Double-sided .ATR files are now supported. Sectors on side 2 are mapped for compatibility with the Atari XF551 disk drive.
        ATX image format
            Enhanced density tracks and extended long sector size encodings are now supported.
            Images are written with 0x5241 ('AR') as the creator.
            Sectors outside of the normal range (1-18 or 1-26) are now preserved.
            Long sector status now matches 1050 behavior.
            Long sectors without CRC errors are now loaded properly.
        SuperCard Pro hardware support
            Fixed a bug in the SuperCard Pro detection logic on Windows where it would try to use the wrong device path if the Virtual COM Port (VCP) driver was configured to use COM10 or higher as the serial port name. This meant that the device could only be used with an explicit path override. The auto-detection code now uses the \\.\ path escape prefix as needed to access the higher-numbered ports.
            Error codes from the SuperCard Pro device are now decoded to readable messages.
            Motor and drive select on the SuperCard Pro are now turned off when an error occurs.
            The number of revolutions for a direct read can now be configured from 1-5 (default 5).
            The direct read code now automatically switches to 8-bit encoding when 16-bit encoding overflows the SCP's memory. This avoids buffer full errors when reading uninitialized or high-density tracks with a full 5 revolutions.
        SCP image format
            Added heuristics to detect 48tpi or 96tpi layouts in SCP images. Due to ambiguities in the SCP image format, this is not guaranteed to work and the scp-ss40, scp-ds40, scp-ss80, and scp-ds80 format IDs have been added to allow forcing one interpretation or the other.
            The disk type for newly written images uses Other 720K instead of Atari FM if 96/135 TPI is used.
            Fixed a bug where the track number in the track header of SCP files was half of what it should have been due to not counting track entries from the skipped side.
            The SCP 1.6+ timestamp and footer are now written to the file.
            The normalized flux timing flag is now set if the flux timing was synthesized from decoded data rather than read from a physical disk.
            Written images are now marked as using Index mode rather than Splice, to more closely match how a8rawconv operates.
            Extended header images are supported (flags bit 6 = 1).
        Encoding
            Improved encoding heuristics to use better track splice points and improve tight track packing.
            Interleave when encoding to flux or converting to ATX can now be controlled with the -i option.

a8rawconv 0.92

        Fixed FDC status flags written into ATX images for long sectors. The CRC error bit is no longer always set and now reflects the status as would be returned by a 1050, which reads the entire sector. The data request (DRQ) flag is also no longer set. 810 emulators must override the flags by clearing the CRC error state and setting the lost data state.
        Bits 2-7 of the sector address size field are now ignored as real FDCs do.
        Fixed crash in macgcr decoder with empty tracks.

a8rawconv 0.91

        Fixed issue causing drift in index mark times when decoding KryoFlux streams. This affected sector positions more noticeably with many revolutions.
        Improved track splice logic when writing tracks to physical disks.

a8rawconv 0.9

        Fixed incorrect ATR header for enhanced density disks.
        Fixed auto decoder selection to choose GCR decoder for Apple II disk formats.
        Tweaked Apple II GCR decoder.
        Added distance restrictions to IDAM/DAM matching to handle IDAM/IDAM/DAM/DAM pattern from interleaved sectors, since real FDCs will reject a DAM that is too close or too far away from the IDAM.
        ATX writer now warns if the disk being encoded is a medium/enhanced or double density disk, which cannot be encoded in ATX.
        Fixed encoding of ATX sector flag bits 5 and 6, which was incorrectly using 810-style signaling; bit 6 is now written properly as an extended data indicator and bit 5 alone reports the DAM.
        Added -tpi switch to read KryoFlux raw stream sets created with a 48 TPI drive.
        Added -I switch for easier checking of Apple II text data, which has the high bit set by default.

Acknowledgments

The author would like to thank Jim Drew for providing SuperCard Pro hardware and support, and fellow AtariAge board members such as Farb for testing, encouragement, and suggestions.
License

a8rawconv is released under the GNU General Public License, version 2 or later. A copy of the license is included in the file COPYING, and the source code is included within the same archive.

While the converter is licensed under the GNU GPL, the converter does not put any part of itself into disk images processed by the converter. Therefore, there are no restrictions on the license of any data or software on disks processed by the converter, nor does processing disks through the converter impose any licensing restrictions on the disk image or data/software within that disk image.
