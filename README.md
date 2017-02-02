# node-video-lib

[![Build Status](https://travis-ci.org/pipll/node-video-lib.svg?branch=master)](https://travis-ci.org/pipll/node-video-lib) [![npm](https://img.shields.io/npm/v/node-video-lib.svg)](https://www.npmjs.com/package/node-video-lib) [![Code Climate](https://codeclimate.com/github/pipll/node-video-lib/badges/gpa.svg)](https://codeclimate.com/github/pipll/node-video-lib) ![GitHub license](https://img.shields.io/github/license/pipll/node-video-lib.svg)

Node.js Video Library.

## Limitations

**This library works only with MP4 video files encoded using H.264 video codec and AAC audio codec.** 

## Installation

```
npm install node-video-lib
```

## Usage

### Parse MP4 file

```javascript
const fs = require('fs');
const MP4Parser = require('node-video-lib').MP4Parser;

fs.open('/path/to/file.mp4', 'r', function(fd) {
    try {
        let movie = MP4Parser.parse(fd);
        // Work with movie
        console.log('Duration:', movie.relativeDuration());
    } catch (ex) {
        console.error('Error:', ex);
    } finally {
        fs.close(fd);
    }
});
```

### Create MPEG-TS chunks

```javascript
const fs = require('fs');
const MP4Parser = require('node-video-lib').MP4Parser;
const FragmentListBuilder = require('node-video-lib').FragmentListBuilder;
const HLSPacketizer = require('node-video-lib').HLSPacketizer;

fs.open('/path/to/file.mp4', 'r', function(fd) {
    try {
        let movie = MP4Parser.parse(fd);
        let fragmentList = FragmentListBuilder.build(movie, 5);
        for (let i = 0; i < fragmentList.count(); i++) {
            let buffer = HLSPacketizer.packetize(fragmentList.get(i), fd);
            // Now buffer contains MPEG-TS chunk
        }
    } catch (ex) {
        console.error('Error:', ex);
    } finally {
        fs.close(fd);
    }
});
```

## Classes

### MP4Parser

A tool for parsing MP4 video files.

```javascript
const MP4Parser = require('node-video-lib').MP4Parser;
```

Methods:

* **parse(fd)** - Parse MP4 file
    * **fd** *\<Integer\>* - File descriptor
    * Return: [*\<Movie\>*](#movie)

### HLSPacketizer

A tool for creating MPEG-TS chunks.

```javascript
const HLSPacketizer = require('node-video-lib').HLSPacketizer;
```

Methods:

* **packetize(fragment, fd)** - Create MPEG-TS chunk from movie fragment
    * **fragment** [*\<Fragment\>*](#fragment) - Movie fragment
    * **fd** *\<Integer\>* - File descriptor
    * Return: [*\<Buffer\>*](https://nodejs.org/api/buffer.html)

### FragmentListBuilder

A tool for splitting the movie into a list of fragments.

```javascript
const FragmentListBuilder = require('node-video-lib').FragmentListBuilder;
```

Methods:

* **build(movie, fragmentDuration)** - Split the movie to a list of fragments with an appropriate duration
    * **movie** [*\<Movie\>*](#movie) - Fragment duration
    * **fragmentDuration** *\<Integer\>* - Fragment duration
    * Return: [*\<FragmentList\>*](#fragmentlist)

### FragmentReader

A tool for reading samples data of the given movie fragment.

```javascript
const FragmentReader = require('node-video-lib').FragmentReader;
```

Methods:

* **readSamples(fragment, fd)** - Read samples data
    * **fragment** [*\<Fragment\>*](#fragment) - Movie fragment
    * **fd** *\<Integer\>* - File descriptor
    * Return: *\<Array\>* Array of buffers

### Movie

A movie class

```javascript
const Movie = require('node-video-lib').Movie;
```

Properties:

* **duration** *\<Integer\>* - Movie duration
* **timescale** *\<Integer\>* - Movie timescale
* **tracks** *\<Array\>* - List of movie tracks

Methods:

* **relativeDuration()** - Movie duration in seconds
    * Return: *\<Number\>*
* **resolution()** - Video resolution
    * Return: *\<String\>*
* **addTrack(track)** - Add a track to the tracks list
    * **track** *\<Track\>* - Track
* **videoTrack()** - Get the first video track
    * Return: *\<VideoTrack\>*
* **audioTrack()** - Get the first audio track
    * Return: *\<AudioTrack\>*
* **samples()** - Get a list of movie samples ordered by relative timestamp
    * Return: *\<Array\>*

### FragmentList

A list of movie fragments class.

```javascript
const FragmentList = require('node-video-lib').FragmentList;
```

Properties:

* **fragmentDuration** *\<Integer\>* - Target fragment duration
* **duration** *\<Integer\>* - Movie duration
* **timescale** *\<Integer\>* - Movie timescale
* **videoExtraData** [*\<Buffer\>*](https://nodejs.org/api/buffer.html) - Video codec information content
* **audioExtraData** [*\<Buffer\>*](https://nodejs.org/api/buffer.html) - Audio codec information content
* **width** *\<Integer\>* - Video width
* **height** *\<Integer\>* - Video height

Methods:

* **relativeDuration()** - Movie duration in seconds
    * Return: *\<Number\>*
* **count()** - Fragments count
    * Return: *\<Integer\>*
* **get(index)** - Get fragment by index
    * Return: [*\<Fragment\>*](#fragment)

### Fragment

A movie fragment class

```javascript
const Fragment = require('node-video-lib').Fragment;
```

Properties:

* **timestamp** *\<Integer\>* - Fragment timestamp
* **duration** *\<Integer\>* - Fragment duration
* **timescale** *\<Integer\>* - Fragment timescale
* **videoExtraData** [*\<Buffer\>*](https://nodejs.org/api/buffer.html) - Video codec information content
* **audioExtraData** [*\<Buffer\>*](https://nodejs.org/api/buffer.html) - Audio codec information content
* **samples** *\<Array\>* - List of fragment samples

Methods:

* **relativeTimestamp()** - Fragment timestamp in seconds
    * Return: *\<Number\>*
* **relativeDuration()** - Fragment duration in seconds
    * Return: *\<Number\>*
* **readSamples(fd)** - Read samples content
    * **fd** *\<Integer\>* - File descriptor

### Track

A general track class

```javascript
const Track = require('node-video-lib').Track;
```

Properties:

* **duration** *\<Integer\>* - Track duration
* **timescale** *\<Integer\>* - Track timescale
* **extraData** [*\<Buffer\>*](https://nodejs.org/api/buffer.html) - Codec information content
* **samples** *\<Array\>* - List of track samples

Methods:

* **relativeDuration()** - Track duration in seconds
    * Return: *\<Number\>*
* **addSample(sample)** - Add a sample to the samples list
    * **sample** *\<Sample\>* - Sample

### AudioTrack

An audio track class. Extends the general track class

```javascript
const AudioTrack = require('node-video-lib').AudioTrack;
```

Properties:

* **channels** *\<Integer\>* - Number of audio channels
* **sampleRate** *\<Integer\>* - Audio sample rate
* **sampleSize** *\<Integer\>* - Audio sample size

### VideoTrack

A video track class. Extends the general track class

```javascript
const AudioTrack = require('node-video-lib').AudioTrack;
```

Properties:

* **width** *\<Integer\>* - Video width
* **height** *\<Integer\>* - Video height

Methods:

* **resolution()** - Video resolution
    * Return: *\<String\>*

### Sample

A general video sample class

```javascript
const Sample = require('node-video-lib').Sample;
```

Properties:

* **timestamp** *\<Integer\>* - Sample timestamp
* **timescale** *\<Integer\>* - Sample timescale
* **size** *\<Integer\>* - Sample size
* **offset** *\<Integer\>* - Sample offset in the file

Methods:

* **relativeTimestamp()** - Sample timestamp in seconds
    * Return: *\<Number\>*

### AudioSample

An audio sample class. Extends the general sample class

```javascript
const AudioSample = require('node-video-lib').AudioSample;
```

### VideoSample

A video sample class. Extends the general sample class

```javascript
const VideoSample = require('node-video-lib').VideoSample;
```

Properties:

* **compositionOffset** *\<Integer\>* - Composition offset
* **keyframe** *\<Boolean\>* - Keyframe flag
