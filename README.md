## npm-audio-modules

Personal notes on various audio modules in npm/javascript.


### node microphone modules
`mic`, `node-microphone`, and `node-record-lpcm16` are very similar in that they wrap `sox`, or `arecord`, or `alsa`
and provide a standard node stream.

`microphone` is a wrapper around ALSA and provides mp3 encoding.
I couldn't get it to work at all. seems broken.


### browser microphone modules

`microphone-stream` provides a node compatible stream from a `getUserMedia()` object.
It's nice being able to use node streams in the browser, but it also makes your front end bundle
pretty huge because shimming node modules for the front end results in a big size increase.

`wave-recorder` is a similar module. Also expose node style streams, which have the same pros/cons
as `microphone-stream`


### processing modules

The web Audio API (specifically the `analyser` object) is not available in node. There's a lot of javascript out in
the wild based on this API, which would be really nice to be able to use in node.

`audio-analyser` is an awesome module that bridges this gap. It provides a writable node 
stream. You pipe microphone data in, and then can access the web audio API analyser functions.

https://github.com/shubhamjain/whistlerr is not an npm module, but it provides a way
to take an audio input from the browser and detect when whistles occur.

my fork, which exposes a node module for this: https://github.com/mreinstein/node-whistlerr

I have a PR which makes shubhamjain's module cross platform, waiting for him to merge it so
that I can remove my copy.


### rendering modules

`audio-render` renders audio waveforms, and it works in both node and browser environments.
Though I find the browser version is too bloated due to reliance on shimmed node modules.

`audio-waveform` is another waveform renderer that works in the browser and node.
It uses node-style streams, so to use this in the browser you need something like 
`microphone-stream` or `wave-recorder` to provide streamable microphone data.
It's a little hard to use because the examples aren't helpful, but I'm hoping they accept 
my PR which provides node and browser usage examples.

`wavearea` provides a waveform renderer, but it also provides simple cut/copy/paste edit
controls!


### data flow

This 

```
     Node Data Flows
+-----------------------------------------------------------------+

  any of these modules             might .pipe() into any of these
                             +
                +-----+      |
                | mic |      |      +----------------+
                +-----+      |      | audio-analyser |
                             |      +----------------+
                             |
    +-----------------+      |      +--------------+
    | node-microphone |   +----->   | audio-render |
    +-----------------+      |      +--------------+
                             |
                             |      +----------------+
+---------------------+      |      | audio-waveform |
| audio-record-lpcm16 |      |      +----------------+
+---------------------+      |
                             |      +----------------+
                             |      | node-whistlerr |
                             |      +----------------+
                             +



    Browser Data Flows
  +--------------------------------------------------------------+

   any of these modules           might .pipe() into any of these
                             +
  +-------------------+      |      +-----------+
  | microphone-stream |      |      | whistlerr |
  +-------------------+      |      +-----------+
                             |
                             |      +--------------+
      +---------------+   +----->   | audio-render |
      | wave-recorder |      |      +--------------+
      +---------------+      |
                             |      +---------------+
                             |      | audo-waveform |
                             |      +---------------+
                             +

```


