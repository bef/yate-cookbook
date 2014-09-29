---
layout: page
title: Audio
permalink: /audio/
---

## Converting Audio for Yate

Despite its name, the *wave* module (as in `wave/play/...`) does not actually support wave files. It can however process raw 8kHz single channel signed-integer PCM audio data. And this is how to create such files:

    sox sound.wav -t raw -r 8000 -c 1 -e signed-integer sound.slin

I like to wrap things in shell scripts for simple reuse:
`wav2slin.sh:`

```sh
#!/bin/sh
sox $1 -t raw -r 8000 -c 1 -e signed-integer $2
```