# @jarif098/ytdl-core

# Usage

```js
const fs = require('fs');
const ytdl = require('ytdl-core');
// TypeScript: import ytdl from 'ytdl-core'; with --esModuleInterop
// TypeScript: import * as ytdl from 'ytdl-core'; with --allowSyntheticDefaultImports
// TypeScript: import ytdl = require('ytdl-core'); with neither of the above

ytdl('http://www.youtube.com/watch?v=aqz-KE-bpKQ')
  .pipe(fs.createWriteStream('video.mp4'));
```


# API
### ytdl(url, [options])

Attempts to download a video from the given url. Returns a [readable stream](https://nodejs.org/api/stream.html#stream_class_stream_readable). `options` can have the following, in addition to any [`getInfo()` option](#async-ytdlgetinfourl-options) and [`chooseFormat()` option](#ytdlchooseformatformats-options).

#### Event: info
* [`ytdl.videoInfo`](typings/index.d.ts#L235) - Info.
* [`ytdl.videoFormat`](typings/index.d.ts#L32) - Video Format.

Emitted when the video's `info` is fetched, along with the chosen format to download.

#### Event: progress
* `number` - Chunk length in bytes or segment number.
* `number` - Total bytes or segments downloaded.
* `number` - Total bytes or segments.

Emitted whenever a new chunk is received. Passes values describing the download progress.

#### miniget events

All [miniget events](https://github.com/fent/node-miniget#event-redirect) are forwarded and can be listened to from the returned stream.

### Stream#destroy()

Call to abort and stop downloading a video.

### async ytdl.getBasicInfo(url, [options])

Use this if you only want to get metainfo from a video.

### async ytdl.getInfo(url, [options])

Gets metainfo from a video. Includes additional formats, and ready to download deciphered URL. This is what the `ytdl()` function uses internally.

`options` can have the following

* `requestOptions` - Anything to merge into the request options which [miniget](https://github.com/fent/node-miniget) is called with, such as `headers`.
* `requestCallback` - Provide a callback function that receives miniget request stream objects used while fetching metainfo.
* `lang` - The 2 character symbol of a language. Default is `en`.

### ytdl.downloadFromInfo(info, options)

Once you have received metadata from a video with the `ytdl.getInfo` function, you may pass that information along with other options to this function.

### ytdl.chooseFormat(formats, options)

Can be used if you'd like to choose a format yourself. Throws an Error if it fails to find any matching format.

`options` can have the following

* `quality` - Video quality to download. Can be an [itag value](http://en.wikipedia.org/wiki/YouTube#Quality_and_formats), a list of itag values, or one of these strings: `highest`/`lowest`/`highestaudio`/`lowestaudio`/`highestvideo`/`lowestvideo`. `highestaudio`/`lowestaudio` try to minimize video bitrate for equally good audio formats while `highestvideo`/`lowestvideo` try to minimize audio respectively. Defaults to `highest`, which prefers formats with both video and audio.

  A typical video's formats will be sorted in the following way using `quality: 'highest'`
  ```
  itag container quality codecs                 bitrate  audio bitrate
  18   mp4       360p    avc1.42001E, mp4a.40.2 696.66KB 96KB
  137  mp4       1080p   avc1.640028            4.53MB
  248  webm      1080p   vp9                    2.52MB
  136  mp4       720p    avc1.4d4016            2.2MB
  247  webm      720p    vp9                    1.44MB
  135  mp4       480p    avc1.4d4014            1.1MB
  134  mp4       360p    avc1.4d401e            593.26KB
  140  mp4               mp4a.40.2                       128KB
  ```
  format 18 at 360p will be chosen first since it's the highest quality format with both video and audio. If you'd like a higher quality format with both video and audio, see the section on [handling separate streams](#handling-separate-streams).
* `filter` - Used to filter the list of formats to choose from. Can be `audioandvideo` or `videoandaudio` to filter formats that contain both video and audio, `video` to filter for formats that contain video, or `videoonly` for formats that contain video and no additional audio track. Can also be `audio` or `audioonly`. You can give a filtering function that gets called with each format available. This function is given the `format` object as its first argument, and should return true if the format is preferable.
  ```js
  // Example with custom function.
  ytdl(url, { filter: format => format.container === 'mp4' })
  ```
* `format` - Primarily used to download specific video or audio streams. This can be a specific `format` object returned from `getInfo`.
  * Supplying this option will ignore the `filter` and `quality` options since the format is explicitly provided.

```js
// Example of choosing a video format.
let info = await ytdl.getInfo(videoID);
let format = ytdl.chooseFormat(info.formats, { quality: '134' });
console.log('Format found!', format);
```

### ytdl.filterFormats(formats, filter)

If you'd like to work with only some formats, you can use the [`filter` option above](#ytdlurl-options).

```js
// Example of filtering the formats to audio only.
let info = await ytdl.getInfo(videoID);
let audioFormats = ytdl.filterFormats(info.formats, 'audioonly');
console.log('Formats with only audio: ' + audioFormats.length);
```

### ytdl.version

The version string taken directly from the package.json.


# Install

```bash
npm install ytdl-core@latest
```

Or for Yarn users:
```bash
yarn add ytdl-core@latest
```


# Tests
Tests are written with [mocha](https://mochajs.org)

```bash
npm test
```
