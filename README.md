# Panako - Acoustic Fingerprinting <!-- omit in toc -->

## Overview <!-- omit in toc -->

`Panako` is an [acoustic fingerprinting](https://en.wikipedia.org/wiki/Acoustic_fingerprint) system. The system is able to extract fingerprints from an audio stream, and either store those fingerprints in a database, or find a match between the extracted fingerprints and stored fingerprints.  Several acoustic fingerprinting algorithms are implemented within `Panako`. The main algorithm, the `Panako` algorithm, has the feature that audio queries can be identified reliably and quickly even if they has been sped up, time stretched or pitch shifted with respect to the reference audio. The main focus of `Panako` is to serve as a demonstration of the `Panako` algorithm. Other acoustic fingerprinting schemes are implemented to serve as a baseline for comparison. More information can be found in the [article about Panako](http://www.terasoft.com.tw/conf/ismir2014/proceedings/T048_122_Paper.pdf).

`Panako` has also uses for synchronization of data-streams. For those applications please see the article titled [Synchronizing Multimodal Recordings Using Audio-To-Audio Alignment](http://0110.be/posts/Synchronizing_Multimodal_Recordings_Using_Audio-To-Audio_Alignment_-_In_Journal_on_Multimodal_User_Interfaces).

Please be aware of the patents [US7627477 B2](https://www.google.com/patents/US7627477) and [US6990453](https://www.google.com/patents/US6990453) and [perhaps others](http://www.shazam.com/music/web/productfeatures.html?id=1284). They describe techniques used in some algorithms implemented within `Panako`. The patents limit the use of some algorithms  under various conditions and for several regions. Please make sure to consult your intellectual property rights specialist if you are in doubt about these restrictions. If these restrictions apply, respect the patent holders rights. The first aim of `Panako` is to serve as a research platform to experiment with and improve fingerprinting algorithms.

This document covers installation, usage and configuration of `Panako`.

The `Panako` source code is licensed under the [GNU Affero General Public License](https://www.gnu.org/licenses/agpl-3.0.html).

---

- [Why Panako?](#why-panako)
- [Getting started](#getting-started)
- [Panako Usage](#panako-usage)
  - [Store Fingerprints - **panako store**](#store-fingerprints---panako-store)
  - [Query for Matches - **panako query**](#query-for-matches---panako-query)
  - [Monitor Stream for Matches - **panako monitor**](#monitor-stream-for-matches---panako-monitor)
  - [Print Storage Statistics - **panako stats**](#print-storage-statistics---panako-stats)
  - [Print Configuration - **panako config**](#print-configuration---panako-config)
  - [Resolve an identifier for a filename - **panako resolve**](#resolve-an-identifier-for-a-filename---panako-resolve)
  - [Browse Fingerprints - **panako browser**](#browse-fingerprints---panako-browser)
  - [Synchronize media files - **panako sync**](#synchronize-media-files---panako-sync)
  - [Synchronize Sensor Streams with SyncSink - **panako syncsink**](#synchronize-sensor-streams-with-syncsink---panako-syncsink)
  - [Compare the structure of media files - **panako compare**](#compare-the-structure-of-media-files---panako-compare)
  - [Run the REST webservice - **panako server**](#run-the-rest-webservice---panako-server)
  - [Query the REST service - **panako client**](#query-the-rest-service---panako-client)
- [Further Reading](#further-reading)
- [Credits](#credits)
- [Reproduce the ISMIR Paper Results](#reproduce-the-ismir-paper-results)
  - [Requirements](#requirements)
- [Changelog](#changelog)

## Why Panako?

Content based music search algorithms make it possible to identify a small audio fragment in a digital music archive with potentially millions of songs. Current search algorithms are able to respond quickly and reliably on an audio query, even if there is noise or other distortions present. During the last decade they have been used successfully as digital music archive management tools, music identification services for smartphones or for digital rights management.

![Fig 1][fig1]

*Fig 1. General content based audio search scheme.*

Most algorithms, as they are described in the literature, do not allow substantial changes in replay speed. The speed of the audio query needs to be the same as the reference audio for the current algorithms to work. This poses a problem, since changes in replay speed do occur commonly, they are either introduced by accident during an analog to digital conversion or are introduced deliberately.

Analogue physical media such as wax cylinders, wire recordings, magnetic tapes and grammophone records can be digitized at an incorrect or varying playback speed. Even when calibrated mechanical devices are used in a digitization process, the media could already have been recorded at an undesirable speed. To identify duplicates in a digitized archive, a music search algorithm should compensate for changes in replay speed

Next to accidental speed changes, deliberate speed manipulations are sometimes introduced during radio broadcasts: occasionally songs are played a bit faster to fit into a timeslot. During a DJ-set speed changes are almost always present. To correctly identify audio in these cases as well, a music search algorithm robust against pitch shifting, time stretching and speed changes is desired.

The `Panako` algorithm allows such changes while maintaining other desired features as scalability, robustness and reliability. Next to two versions of the `Panako` algorithm (internally identified as CTEQ and NCTEQ), two versions of the algorithm described in [An Industrial-Strength Audio Search Algorithm](http://www.ee.columbia.edu/~dpwe/papers/Wang03-shazam.pdf) are implemented (internally identified as FFT and NFFT). Also the algorithm in [A Robust Audio Fingerprinter Based on Pitch Class Histograms - Applications for Ethnic Music Archives](http://0110.be/files/attachments/415/2012.01.20.fingerprinter.submitted.pdf) is available. To make comparisons between fingerprinting systems easy, researchers are kindly invited to contribute algorithms to the `Panako` project.

Alternative open source music identification systems are [audfprint](http://www.ee.columbia.edu/~dpwe/resources/matlab/audfprint/) and [echoprint](http://echoprint.me/). Alternative systems with similar features are described in [US7627477 B2](https://www.google.com/patents/US7627477) and in [Quad-based Audio Fingerprinting Robust To Time And Frequency Scaling (2014)](http://www.dafx14.fau.de/papers/dafx14_reinhard_sonnleitner_quad_based_audio_fingerpr.pdf) by Reinhard Sonnleitner and Gerhard Widmer.

---

## Getting started

- `src` contains the Java source files.
- `lib` contains the dependencies (JAR-files).
- `build` contains an ant build file to compile `Panako` and the documentation. Use `ant release` to build the source and compile `Panako`.
- `doc` contains documentation for the source files, generated with `javadoc`.

To compile `Panako`, the JDK 8 or later is required.

`Panako` uses the [Apache Ant](https://ant.apache.org/) build system. Install it on your system.

NOTE: You may need to run the ant commands with elevated privileges (sudo)

```bash
sudo apt-get install default-jdk ant libav-tools # install JDK, ant libav-tools (on Ubuntu).
sudo apt-get install ruby # needed for documentation generation
cd ./build
ant # builds the core Panako library
ant install # installs Panako in the /opt/panako directory
ant doc # creates the documentation in Panako/doc
```

If you want to call `panako` without having to refer to the local executable:

```bash
cp ../doc/panako /usr/bin
```

This command copies the startup script in `doc/panako` to a directory in your path. The script allows for easy access to `Panako`'s functionality. If this does not work for you, you can still call `Panako` using `java -jar /opt/panako/panako.jar [..args]`.

`Panako` decodes audio using by calling `ffmpeg`, an utility for decoding audio. If it is not present on your system, `Panako` automatically tries to download a suitable version. If this attempt fails, however, it needs to be installed manually and should be available on your systems path. On a Debian like system:

```bash
apt-get install ffmpeg
```

`Panako` calls a sub-process `ffmpeg` and reads decoded PCM audio samples via a pipe. By default `Panako` calls `ffmpeg`. Alternatively, `Panako` can be configured to use any utility (like `avconv`) that can pipe decoded audio streams in the format PCM, one channel, 16bit per sample.

---

## Panako Usage

`Panako` provides a command line interface, it can be called using `panako subapplication [argument...]`. For each task `Panako` has to perform, there is a subapplication. There are subapplications to store fingerprints, query audio fragments, monitor audio streams, and so on. Each subapplication has its own list of arguments, options, and output format that define the behavior of the subapplication.

To save some keystrokes the name of the subapplication can be shortened using a unique prefix. For example `panako m file.mp3` is expanded to `panako monitor file.mp3`. Since both `stats` and `store` are valid subapplications the `store` call can be shortened to `panako sto *.mp3`, `panako s *.mp3` gives an invalid application message. A [trie](https://en.wikipedia.org/wiki/Trie) is used to find a unique prefix.

What follows is a list of those subapplications, their arguments, and respective goal.

### Store Fingerprints - **panako store**

The `store` instruction extracts fingerprints from audio tracks and stores those in the datastorage. The command expects a list of audio files, video files or a text file with a list of file names.

```bash
#Audio is converted automatically
panako store audio.mp3 audio.ogg audio.flac audio.wav

#The first audio stream of video containers formats is used.
panako store audio.mpc audio.mp4 audio.acc audio.avi audio.ogv audio.wma

#Glob characters can be practical
panako store */*.mp3

#A text file with audio files can be used as well
#The following searches for all mp3's (recursively) and
#stores them in a text file
find . -name '*.mp3' > list.txt
#Iterate the list
panako store list.txt
```

### Query for Matches - **panako query**

The `query` command extracts fingerprints from a short audio frament and tries to match the fingerprints with the database.

```bash
panako query short_audio.mp3
```

### Monitor Stream for Matches - **panako monitor**

The `query` command extracts fingerprints from a short audio frament and tries to match the fingerprints with the database. The incoming audio is, by default, chopped in parts of 25s with an overlap of 5s. So every `25-5=20s` the database is queried and a result is printed to the command line.

```bash
panako monitor [short_audio.mp3]
```

If no audio file is given, the default microphone is used as input.

### Print Storage Statistics - **panako stats**

The `stats` command prints statistics about the stored fingerprints and the number of audio fragments. If an integer argument is given it keeps printing the stats every x seconds.

```bash
panako stats # print stats once
panako stats 20 # print stats every 20s
```

### Print Configuration - **panako config**

The `config` subapplication prints the configuration currently in use.

```bash
panako config
```

To override configuration values there are two options. The first option is to create a configuration file, by default at the following location: `/opt/config.properties`. The configuration file is a properties text file. An commented configuration file should be included in the doc directory at `doc/config.properties`.

The second option to override configuration values is by adding them to the arguments of the command line call as follows:

```bash
panako subapplication CONFIG_KEY=value
```

For example, if you do not want to check for duplicate files while building a fingerprint database the following can be done:

```bash
panako store file.mp3 CHECK_FOR_DUPLICATES=FALSE
```

The configuration values provided as a command line argument have priority over the ones in the configuration file. If there is no value configured a default is used automatically. To find out which configuration options are available and their respective functions, consult the documented example configuration file `doc/config.properties`.

### Resolve an identifier for a filename - **panako resolve**

This application simply returns the identifier that is used internally for a filename. The following call returns for example `54657653`:

```bash
panako resolve test.mp3
```

The internal identifiers are currently defined using integers.

### Browse Fingerprints - **panako browser**

Shows a swing user interface with a spectrogram and dectected event points and fingerprints. It can be used
to check the effect of configuration parameters on fingerprint detection and to optimize configuration.

### Synchronize media files - **panako sync**

This operation extracts fingerprints from a reference media file and returns how much time offset there is to other files with similar audio. This is practical to synchronize video streams with similar audio or recordings of the same performance from several microphones.

The synchonize option uses a two stage algorithm. First fingerprints are extracted from the media files. With these it is checked if the media files contain similar audio. If they do, a rough offset is determined (e.g. with 32ms accuracy). Subsequently crosscovariance of one audio frame is used to improve the offset estimation. This offset is returned.

```bash
panako sync reference.avi other.mkv other.mp3
```

### Synchronize Sensor Streams with SyncSink - **panako syncsink**

This command starts a Swing user interface to synchronize several audio/video and data files. The aim is to make it easy to synchronize various media files and to allow synchronization of sensor data streams. Syncronization of sensor streams can be done effectively if for each sensor stream there is a corresponding synchronized ambient audio recording. The problem of synchronizing sensor streams is then reduced to audio-to-audio alignment. The SyncSink facilitates this type of data-stream synchronization.

The synchonize option uses a two stage algorithm. First fingerprints are extracted from the media files. With these it is checked if the media files contain similar audio. If they do, a rough offset is determined (e.g. with 32ms accuracy). Subsequently crosscovariance of one audio frame is used to improve the offset estimation. This offset is returned.

![Fig 2][fig2]

*Fig 2. SyncSink user interface.*

To use the application start it with `panako syncsink`. Subsequently add various audio or video files using drag and drop. If the same audio is found in the various media files a timebox plot appears, as in the screenshot below. To add corresponding data-files click one of the boxes on the timeline and choose a data file that is synchronized with the audio. The datafile should be a CSV-file. The separator should be ',' and the first column should contain a timestamp in fractional seconds. After pressing `Sync` a new CSV-file is created with the first column containing correctly shifted time stamps. If this is done for multiple files, a synchronized sensor-stream is created. Also, [ffmpeg](https://www.ffmpeg.org/) commands to syncronize the media files themselves are printed to the command line.

It is also possible to start the application with media streams immediately pressent. The first file is considered to be the reference stream.

```bash
panako syncsink reference.avi first.mkv second.mp3 third.wav
```

By default `Panako` is configured to synchronize music and to minimize the number of event points and fingerprints per second. This reduces the storage requirements and processing needs. However, it can be of help to change the default configuration to synchronize sparse audio or speech. In such case try to start `panako` with the following set of configuration values:

```bash
panako syncsink \
  SYNC_MIN_ALIGNED_MATCHES=4 \
  NFFT_EVENT_POINT_MIN_ENERGY_RATIO_THRESHOLD=0.10 \
  NFFT_EVENT_POINT_MAX_ENERGY_RATIO_THRESHOLD=0.90 \
  NFFT_EVENT_POINT_MIN_ENERGY=0.01 \
  NFFT_MAX_FINGERPRINTS_PER_EVENT_POINT=7
```

### Compare the structure of media files - **panako compare**

This operation extracts fingerprints from a reference media file and searches for the fingerprints in another media file. The aim is to identify the parts of audio that are present in both input files. If only one audio file is given, a self similarity matrix is constructed. Using the matrix it becomes clear when the exact same audio is repeated within the same song. It can be used to automatically detect where cuts have been made in a long original album edit to end up with a shorter radio edit of the same song. Another use case is musical structure analysis, this works especially well in electronic music. To call the application:

```bash
panako compare audio.wav > self_similarity.csv
```

```bash
panako compare original_edit.wav radio_edit.wav > cuts_where.csv
```

The output is a CSV file with millisecond values. When only one file is given, the diagonal is detected in a self similarity matrix and the output could look like below, if there is a repetiion of the audio between 3s-3.5s at 90-90.5s. The data is easy to visualize in a spreadsheet application.

```csv
10,10
15,15
20,20
...
3000,3000,90000
3010,3010,90010
3500,3500,90500
...
4000,4000
4010,4010
...
```

### Run the REST webservice - **panako server**

`Panako` contains a [lightweight HTTP server](http://acme.com/java/software/Acme.Serve.Serve.html) to provide a REST webservice. The REST webservice provides two methods `/v1.0/match` and `/v1.0/metadata` to query for a match and to fetch metadata
for a match respectively. The server can be configured using the `HTTP_SERVER_PORT` configuration setting.

```bash
panako server
```

### Query the REST service - **panako client**

`Panako` contains functionality to send match requests to the REST webservice.

```bash
panako client "localhost:8080" test.mp3
```

---

## Further Reading

Some relevant reading material about acoustic fingerprinting. The order gives an idea of relevance to the `Panako` project.

- Six, Joren and Leman, Marc [__Panako - A Scalable Acoustic Fingerprinting System Handling Time-Scale and Pitch Modification__](http://www.terasoft.com.tw/conf/ismir2014/proceedings/T048_122_Paper.pdf) (2014)
- Wang, Avery L. __An Industrial-Strength Audio Search Algorithm__ (2003)
- Cano, Pedro and Batlle, Eloi and Kalker, Ton and Haitsma, Jaap __A Review of Audio Fingerprinting__ (2005)
- Six, Joren and Cornelis, Olmo __A Robust Audio Fingerprinter Based on Pitch Class Histograms - Applications for Ethnic Music Archives__ (2012)
- Arzt, Andreas and Bock, Sebastian and Widmer, Gerhard __Fast Identification of Piece and Score Position via Symbolic Fingerprinting__ (2012)
- Fenet, Sebastien and Richard, Gael and Grenier, Yves __A Scalable Audio Fingerprint Method with Robustness to Pitch-Shifting__ (2011)
- Ellis, Dan and Whitman, Brian and Porter, Alastair __Echoprint - An Open Music Identification Service__ (2011)
- Sonnleitner, Reinhard  and Widmer, Gerhard __Quad-based Audio Fingerprinting Robust To Time And Frequency Scaling__ (2014)
- Sonnleitner, Reinhard  and Widmer, Gerhard [__Robust Quad-Based Audio Fingerprinting__](http://dx.doi.org/10.1109/TASLP.2015.2509248) (2015)

---

## Credits

The `Panako` software was developed at [IPEM, Ghent University](http://www.ipem.ugent.be/) by Joren Six.

Some parts of `Panako` were inspired by the [Robust Landmark-Based Audio Fingerprinting Matlab implementation by Dan Ellis](http://www.ee.columbia.edu/~dpwe/resources/matlab/fingerprint/).

If you use `Panako` for research purposes, please cite the following work:

```latex
@inproceedings{six2014panako,
  author      = {Joren Six and Marc Leman},
  title       = {{Panako - A Scalable Acoustic Fingerprinting System Handling Time-Scale and Pitch Modification}},
  booktitle   = {{Proceedings of the 15th ISMIR Conference (ISMIR 2014)}},
  year        =  2014
}
```

If you use the synchronization algorithms for research purposes, please cite the following work:

```latex
@article{six2015multimodal,
  author      = {Joren Six and Marc Leman},
  title       = {{Synchronizing Multimodal Recordings Using Audio-To-Audio Alignment}},
  issn        = {1783-7677},
  volume      = {9},
  number      = {3},
  pages       = {223-229},
  doi         = {10.1007/s12193-015-0196-1},
  journal     = {{Journal of Multimodal User Interfaces}},
  publisher   = {Springer Berlin Heidelberg},
  year        = 2015
}
```

---

## Reproduce the ISMIR Paper Results

The directory `doc/Reproducibility` contains scripts to reproduce the result found in the
[Panako ISMIR 2014 paper](http://www.terasoft.com.tw/conf/ismir2014/proceedings/T048_122_Paper.pdf). The scripts follow this procedure:

- The Jamendo dataset is downloaded.
- The fingerprints are extracted and stored for each file in the data set.o download an openly available dataset
- Query files are created from the Jamendo data set.
- `Panako` is queried for each query file, results are logged
- Results are parsed.

### Requirements

To run the scripts, a UNIX like system with following utilities is required:

- wget: an utility to download files
- [SoX](http://sox.sourceforge.net/), inlcuding support for MP3 and GSM encoding.
- bash: the bash shell
- [Ruby](https://www.ruby-lang.org/en/) The ruby programming language runtime environment.
- Unzip: to unzip the metadata file.

Also the `panako` software is needed. Please see above to install the `Panako`.
The configuration used during the test is also included in the `doc/Reproducibility` directory.

If all requirements are met, running the test is done using:

```bash
bash run_experiment.bash
```

---

## Changelog

### Version 1.5 (2016-12-08) <!-- omit in toc -->

Improvements to SyncSink. The cross correlation now behaves well also in edge cases. The Sync code has been simplified to allow maintenance. Included unit tests. Updated to TarsosDSP version 2.4.

### Version 1.6 (2017-03-17) <!-- omit in toc -->

This release adds a simplified version of chromaprint and an implementation of ['A Highly Robust Audio Fingerprinting System' by Haitsma and Kalker](http://www.ismir2002.ismir.net/proceedings/02-FP04-2.pdf)

[fig1]: data:image/png;base64,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

[fig2]: data:image/png;base64,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
