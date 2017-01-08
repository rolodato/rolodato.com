---
layout: post
title: Korg EMX Sample Pack
---

Download all 207 PCM drum samples from a Korg EMX in 16-bit stereo WAV format:

* [0% tube gain](/korg-emx-sample-pack.zip)
* [50% tube gain](/korg-emx-sample-pack-50tube.zip)
* [100% tube gain](/korg-emx-sample-pack-100tube.zip)

I recorded a long file of manually playing each sample one after the other, with some silence in between to ensure I didn't cut off the samples early.
The packs with tube gain were recorded [using a ChucK script to automatically switch between samples](https://github.com/rolodato/sample-pack-generator), with the tubes already warmed up for over an hour.
The samples were played with the following settings on the EMX:

* Level: 127
* Pan: center
* Pitch: 0
* EG time: 127
* Amp EG: non-decay
* FX send: disabled

This entire recording was normalized at 0 dB to ensure a consistent sample level.

I then imported this file into [Audacity](http://www.audacityteam.org) and used the [Sound Finder and Export Multiple features](http://manual.audacityteam.org/man/splitting_a_recording_into_separate_tracks.html) to automatically export one file per sample, using the following settings:

* Treat audio below this level as silence: -40 dB (-30 dB when using tubes)
* Minimum duration of silence between sounds: 0.2 s
* Label starting point: 0.01 s
* Label ending point: 0.01 s
* Add a label at the end of the track: No

The metadata editor that pops up for each file when exporting [can be disabled](http://manual.audacityteam.org/man/import_export_preferences.html).

Audacity exported files numbered `1.wav` to `207.wav` without leading zeroes, which can mess up alphabetical file ordering in some cases.
To add these leading zeroes, I used the following bash script:

{% highlight bash %}
for a in [0-9]*.wav; do
    mv $a `printf %03d.%s ${a%.*} ${a##*.}`
done
{% endhighlight %}

Finally, I stripped any unnecessary metadata that Audacity had included in these files:

{% highlight bash %}
mkdir meta
for i in *.wav; do
    ffmpeg -i "$i" -map 0 -map_metadata 0:s:0 -c copy "meta/$i"
done
{% endhighlight %}
