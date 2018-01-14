# Opus & Wave Recorder

A javascript library to encode the output of Web Audio API nodes in Ogg Opus or WAV format using WebAssembly.
Encoded and muxed audio will be returned as typedArray in `ondataAvailable`.

#### Libraries Used

- Libopus: v1.2.1 compiled with emscripten 1.37.28
- speexDSP: 1.2RC3 compiled with emscripten 1.37.28

#### Required Files

The required files are in the `dist` folder. Unminified files are in `dist/unminified`.


---------
### Usage


---------
#### Constructor

The `Recorder` object is available in the global namespace and supports CommonJS and AMD imports.

```js
var rec = new Recorder([config]);
```
Creates a recorder instance.

- **config** - An optional configuration object (see **config** section below)


---------
#### Config options for OGG OPUS encoder

- **bufferLength**                - (*optional*) The length of the buffer that the internal JavaScriptNode uses to capture the audio. Can be tweaked if experiencing performance issues. Defaults to `4096`.
- **encoderApplication**          - (*optional*) Supported values are: `2048` - Voice, `2049` - Full Band Audio, `2051` - Restricted Low Delay. Defaults to `2049`.
- **encoderBitRate**              - (*optional*) Target bitrate in bits/sec. The encoder selects an application-specific default when this is not specified.
- **encoderComplexity**           - (*optional*) Value between 0 and 10 which determines latency and processing for encoding. `0` is fastest with lowest complexity. `10` is slowest with highest complexity. The encoder selects a default when this is not specified.
- **encoderFrameSize**            - (*optional*) Specifies the frame size in ms used for encoding. Defaults to `20`.
- **encoderPath**                 - (*optional*) Path to `encoderWorker.min.js` or `waveWorker.min.js` worker script. Defaults to `encoderWorker.min.js`
- **encoderSampleRate**           - (*optional*) Specifies the sample rate to encode at. Defaults to `48000`. Supported values are `8000`, `12000`, `16000`, `24000` or `48000`.
- **leaveStreamOpen**             - (*optional*) Keep the stream around when trying to `stop` recording, so you can re-`start` without re-`initStream`. Defaults to `false`.
- **maxBuffersPerPage**           - (*optional*) Maximum number of buffers to use before generating an Ogg page. This can be used to lower the streaming latency. The lower the value the more overhead the ogg stream will incur. Defaults to `40`.
- **mediaTrackConstraints**       - (*optional*) Object to specify [media track constraints](https://developer.mozilla.org/en-US/docs/Web/API/MediaTrackConstraints). Defaults to `true`.
- **monitorGain**                 - (*optional*) Sets the gain of the monitoring output. Gain is an a-weighted value between `0` and `1`. Defaults to `0`
- **numberOfChannels**            - (*optional*) The number of channels to record. `1` = mono, `2` = stereo. Defaults to `1`. Maximum `2` channels are supported.
- **originalSampleRateOverride**  - (*optional*) Override the ogg opus 'input sample rate' field. Google Speech API requires this field to be `16000`.
- **resampleQuality**             - (*optional*) Value between 0 and 10 which determines latency and processing for resampling. `0` is fastest with lowest quality. `10` is slowest with highest quality. Defaults to `3`.
- **streamPages**                 - (*optional*) `dataAvailable` event will fire after each encoded page. Defaults to `false`.


---------
#### Config Options for WAV recorder

- **bufferLength**                - (*optional*) The length of the buffer that the internal JavaScriptNode uses to capture the audio. Can be tweaked if experiencing performance issues. Defaults to `4096`.
- **encoderPath**                 - (*optional*) Path to `encoderWorker.min.js` or `waveWorker.min.js` worker script. Defaults to `encoderWorker.min.js`
- **leaveStreamOpen**             - (*optional*) Keep the stream around when trying to `stop` recording, so you can re-`start` without re-`initStream`. Defaults to `false`.
- **mediaTrackConstraints**       - (*optional*) Object to specify [media track constraints](https://developer.mozilla.org/en-US/docs/Web/API/MediaTrackConstraints). Defaults to `true`.
- **monitorGain**                 - (*optional*) Sets the gain of the monitoring output. Gain is an a-weighted value between `0` and `1`. Defaults to `0`
- **numberOfChannels**            - (*optional*) The number of channels to record. `1` = mono, `2` = stereo. Defaults to `1`. Maximum `2` channels are supported.
- **wavBitDepth**                 - (*optional*) Desired bit depth of the WAV file. Defaults to `16`. Supported values are `8`, `16`, `24` and `32` bits per sample.


---------
#### Instance Methods


```js
rec.initStream()
```

**initStream** will request the user for permission to access the the audio stream. Returns a Promise which resolves the audio stream when it is ready.

```js
rec.pause()
```

**pause** will keep the stream and monitoring alive, but will not be recording the buffers. Will raise the pause event. Subsequent calls to **resume** will add to the current recording.

```js
rec.resume()
```

**resume** will resume the recording if paused. Will raise the resume event.

```js
rec.setMonitorGain( gain )
```

**setMonitorGain** will set the volume on what will be passed to the monitor. Monitor level does not affect the recording volume. Gain is an a-weighted value between `0` and `1`.

```js
rec.start()
```

**start** will initalize the worker and begin capturing audio if the audio stream is ready. Will raise the `start` event when started.

```js
rec.stop()
```

**stop** will cease capturing audio and disable the monitoring and mic input stream. Will request the recorded data and then terminate the worker once the final data has been published. Will raise the `stop` event when stopped.

```js
rec.clearStream()
```

**clearStream** will stop and delete the stream got from `initStream`, you will only ever call this manually if you have `config.leaveStreamOpen` set to `true`.


---------
#### Static Methods

```js
Recorder.isRecordingSupported()
```

Returns a truthy value indicating if the browser supports recording.


---------
#### Callback Handlers

```js
rec.ondataavailable( arrayBuffer )
```
A callback which returns an array buffer of audio data. If `streamPages` is `true`, this will be called with each page of encoded audio.  If `streamPages` is `false`, this will be called when the recording is finished with the complete data.


```js
rec.onpause()
```

A callback which occurs when media recording is paused.

```js
rec.onresume()
```

A callback which occurs when media recording resumes after being paused.


```js
rec.onstart()
```

A callback which occurs when media recording starts.

```js
rec.onstop()
```

A callback which occurs when media recording ends.


---------
### Browser Support

Supported:
- Chrome v58
- Firefox v53
- Microsoft Edge v41
- Opera v44
- macOS Safari v11
- iOS Safari v11

Unsupported:
- IE 11 and below
- iOS 11 Chrome


---------
### Known Issues

- Firefox does not support sample rates above 48000Hz: https://bugzilla.mozilla.org/show_bug.cgi?id=1124981
- macOS Safari v11 does not sample rates above 44100Hz
- macOS Safari v11 native opus playback not yet supported
- iOS Safari v11 native opus playback not yet supported
- Microsoft Edge native opus playback not yet supported


---------
### Building from sources

Prebuilt sources are included in the dist folder. However below are instructions if you want to build them yourself. Opus and speex are compiled without SIMD optimizations. Performace is significantly worse with SIMD optimizations enabled.

Mac: Install autotools using [MacPorts](https://www.macports.org/install.php)
```bash
port install automake autoconf libtool pkgconfig
```

Windows: Install autotools using [MSYS2](http://www.msys2.org/)
```bash
pacman -S make autoconf automake libtool pkgconfig
```

[Inatall Node.js](https://nodejs.org/en/download/)

[Install EMScripten](https://kripken.github.io/emscripten-site/docs/getting_started/downloads.html)

Install npm dependencies:
```bash
npm install
```

checkout, compile and create the dist from sources:
```bash
npm run make
```

Running the unit tests:
```bash
npm test
```

Clean the dist folder and git submodules:
```bash
make clean
```

