# Live Coding: Media Player System

## Goal
Build a media player system demonstrating **Factory Method**, **Facade**, and **Iterator** patterns — plus UML class diagram design on the whiteboard.

---

## Part A: UML Class Diagram Design (on paper/whiteboard)

## Step 1: Identify the classes

Draw four class boxes:

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ MediaPlayer  │  │    Song      │  │   Playlist   │  │  AudioCodec  │
└──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘
```

> **Discussion:** What is each class responsible for? What data does each one own?

## Step 2: Add attributes and methods

```
┌─────────────────────────┐
│      MediaPlayer        │
├─────────────────────────┤
│ - current_song: Song    │
│ - playlist: Playlist    │
│ - volume: int           │
├─────────────────────────┤
│ + play(song): void      │
│ + pause(): void         │
│ + skip(): void          │
└─────────────────────────┘

┌─────────────────────────┐
│         Song            │
├─────────────────────────┤
│ - title: str            │
│ - artist: str           │
│ - duration: int         │
│ - file_path: str        │
├─────────────────────────┤
│ + __str__(): str        │
└─────────────────────────┘

┌─────────────────────────┐
│       Playlist          │
├─────────────────────────┤
│ - name: str             │
│ - songs: list[Song]     │
├─────────────────────────┤
│ + add(song): void       │
│ + __iter__(): Iterator  │
│ + __len__(): int        │
└─────────────────────────┘

┌─────────────────────────┐
│   AudioCodec «ABC»      │
├─────────────────────────┤
│                         │
├─────────────────────────┤
│ + decode(filename): str │
└─────────────────────────┘
```

> **Discussion:** Why is `AudioCodec` abstract? What concrete classes will implement it?

## Step 3: Draw relationships

```
MediaPlayer ◆── Playlist          composition (1..1)
Playlist    ◇── Song              aggregation (0..*)
MediaPlayer ..> AudioCodec        dependency (uses)
```

- **Composition** (filled diamond): MediaPlayer *owns* its Playlist — if the player is destroyed, so is the playlist.
- **Aggregation** (hollow diamond): Playlist *contains* Songs, but songs exist independently.
- **Dependency** (dashed arrow): MediaPlayer *uses* AudioCodec to decode files, but doesn't own one permanently.

## Step 4: Add visibility and multiplicity

```
MediaPlayer "1" ◆──── "1" Playlist
Playlist    "1" ◇──── "0..*" Song
MediaPlayer "1" ..>   "0..1" AudioCodec
```

Visibility markers on the diagram:
- `+` public: `play()`, `pause()`, `skip()`, `add()`, `decode()`
- `-` private: `current_song`, `volume`, `songs`, `title`, `artist`
- `#` protected: (none in this design — discuss when you'd use it)

> **Key point:** The UML diagram is a blueprint. Now we'll implement three patterns that make this design work.

---

## Part B: Factory Method — AudioCodec

## Step 5: Create the AudioCodec abstraction

```python
from abc import ABC, abstractmethod


class AudioCodec(ABC):
    @abstractmethod
    def decode(self, filename):
        pass
```

> **Discussion:** Why an ABC? Because we want to guarantee every codec has a `decode` method. The factory will return different concrete codecs, but the client code doesn't care which one.

## Step 6: Create concrete codecs

```python
class MP3Codec(AudioCodec):
    def decode(self, filename):
        return f"Decoding MP3 audio stream: {filename}"


class WAVCodec(AudioCodec):
    def decode(self, filename):
        return f"Decoding raw WAV data: {filename}"


class FLACCodec(AudioCodec):
    def decode(self, filename):
        return f"Decoding lossless FLAC stream: {filename}"
```

## Step 7: Create the factory

```python
class CodecFactory:
    _codecs = {
        ".mp3": MP3Codec,
        ".wav": WAVCodec,
        ".flac": FLACCodec,
    }

    @classmethod
    def create_codec(cls, file_extension):
        codec_class = cls._codecs.get(file_extension)
        if codec_class is None:
            raise ValueError(f"Unsupported format: {file_extension}")
        return codec_class()
```

> **Discussion:** Why a dictionary instead of if/elif chains? Adding `.ogg` support means adding one line to the dict — no branching logic to modify.

## Step 8: Use the factory

```python
files = ["song.mp3", "track.wav", "album.flac"]

for file in files:
    extension = "." + file.split(".")[-1]
    codec = CodecFactory.create_codec(extension)
    print(codec.decode(file))
```

### Output:
```
Decoding MP3 audio stream: song.mp3
Decoding raw WAV data: track.wav
Decoding lossless FLAC stream: album.flac
```

> **Key point:** The client code never mentions `MP3Codec`, `WAVCodec`, or `FLACCodec` — it asks the factory and gets the right one. Adding a new format requires zero changes to client code.

---

## Part C: Facade — MediaPlayer

## Step 9: Create subsystem classes

```python
class AudioEngine:
    def load(self, filename):
        print(f"  AudioEngine: loading {filename}")

    def play_stream(self):
        print("  AudioEngine: playing audio stream")

    def stop_stream(self):
        print("  AudioEngine: stopping audio stream")


class Equalizer:
    def apply_preset(self, genre):
        print(f"  Equalizer: applying {genre} preset")


class VolumeController:
    def __init__(self):
        self.level = 50

    def set_level(self, level):
        self.level = level
        print(f"  VolumeController: volume set to {level}%")


class DisplayManager:
    def show_now_playing(self, title, artist):
        print(f"  Display: Now playing '{title}' by {artist}")

    def clear(self):
        print("  Display: cleared")
```

> **Discussion:** Each subsystem does one thing. But playing a song requires coordinating all four. Should client code know about all of them?

## Step 10: Create the facade

```python
class MediaPlayerFacade:
    def __init__(self):
        self.engine = AudioEngine()
        self.equalizer = Equalizer()
        self.volume = VolumeController()
        self.display = DisplayManager()

    def play(self, song):
        print(f"▶ Playing: {song.title}")
        self.engine.load(song.file_path)
        self.equalizer.apply_preset("pop")
        self.volume.set_level(70)
        self.engine.play_stream()
        self.display.show_now_playing(song.title, song.artist)

    def pause(self):
        print("⏸ Paused")
        self.engine.stop_stream()

    def skip(self, song):
        print(f"⏭ Skipping to: {song.title}")
        self.engine.stop_stream()
        self.play(song)
```

## Step 11: Client code uses only the facade

```python
class Song:
    def __init__(self, title, artist, duration, file_path):
        self.title = title
        self.artist = artist
        self.duration = duration
        self.file_path = file_path


song1 = Song("Bohemian Rhapsody", "Queen", 354, "bohemian.mp3")
song2 = Song("Stairway to Heaven", "Led Zeppelin", 482, "stairway.flac")

player = MediaPlayerFacade()
player.play(song1)
print()
player.pause()
print()
player.skip(song2)
```

### Output:
```
▶ Playing: Bohemian Rhapsody
  AudioEngine: loading bohemian.mp3
  Equalizer: applying pop preset
  VolumeController: volume set to 70%
  AudioEngine: playing audio stream
  Display: Now playing 'Bohemian Rhapsody' by Queen

⏸ Paused
  AudioEngine: stopping audio stream

⏭ Skipping to: Stairway to Heaven
  AudioEngine: stopping audio stream
▶ Playing: Stairway to Heaven
  AudioEngine: loading stairway.flac
  Equalizer: applying pop preset
  VolumeController: volume set to 70%
  AudioEngine: playing audio stream
  Display: Now playing 'Stairway to Heaven' by Led Zeppelin
```

> **Key point:** The client calls `play()`, `pause()`, `skip()` — three methods. Behind the facade, four subsystems coordinate. The complexity is hidden, not eliminated.

---

## Part D: Iterator — Playlist

## Step 12: Create the Song class

```python
class Song:
    def __init__(self, title, artist, duration):
        self.title = title
        self.artist = artist
        self.duration = duration

    def __str__(self):
        minutes = self.duration // 60
        seconds = self.duration % 60
        return f"{self.title} - {self.artist} ({minutes}:{seconds:02d})"
```

```python
song = Song("Yesterday", "The Beatles", 125)
print(song)
```

### Output:
```
Yesterday - The Beatles (2:05)
```

## Step 13: Create Playlist with the iterator protocol

```python
class Playlist:
    def __init__(self, name):
        self.name = name
        self.songs = []
        self._index = 0

    def add(self, song):
        self.songs.append(song)

    def __len__(self):
        return len(self.songs)

    def __iter__(self):
        self._index = 0
        return self

    def __next__(self):
        if self._index >= len(self.songs):
            raise StopIteration
        song = self.songs[self._index]
        self._index += 1
        return song
```

## Step 14: Use in a for loop

```python
playlist = Playlist("Classic Rock")
playlist.add(Song("Bohemian Rhapsody", "Queen", 354))
playlist.add(Song("Stairway to Heaven", "Led Zeppelin", 482))
playlist.add(Song("Hotel California", "Eagles", 391))

print(f"Playlist: {playlist.name} ({len(playlist)} songs)")
print()

for song in playlist:
    print(f"  ♪ {song}")
```

### Output:
```
Playlist: Classic Rock (3 songs)

  ♪ Bohemian Rhapsody - Queen (5:54)
  ♪ Stairway to Heaven - Led Zeppelin (8:02)
  ♪ Hotel California - Eagles (6:31)
```

> **Discussion:** Python's `for` loop calls `__iter__()` to get the iterator, then `__next__()` repeatedly until `StopIteration` is raised. We implemented the protocol — Python does the rest.

## Step 15: Add a shuffle iterator variant

```python
import random


class ShuffleIterator:
    def __init__(self, playlist):
        self.songs = list(playlist.songs)
        random.shuffle(self.songs)
        self._index = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self._index >= len(self.songs):
            raise StopIteration
        song = self.songs[self._index]
        self._index += 1
        return song
```

```python
playlist = Playlist("Classic Rock")
playlist.add(Song("Bohemian Rhapsody", "Queen", 354))
playlist.add(Song("Stairway to Heaven", "Led Zeppelin", 482))
playlist.add(Song("Hotel California", "Eagles", 391))

print("Normal order:")
for song in playlist:
    print(f"  ♪ {song}")

print()

print("Shuffle order:")
for song in ShuffleIterator(playlist):
    print(f"  ♪ {song}")
```

### Output:
```
Normal order:
  ♪ Bohemian Rhapsody - Queen (5:54)
  ♪ Stairway to Heaven - Led Zeppelin (8:02)
  ♪ Hotel California - Eagles (6:31)

Shuffle order:
  ♪ Hotel California - Eagles (6:31)
  ♪ Bohemian Rhapsody - Queen (5:54)
  ♪ Stairway to Heaven - Led Zeppelin (8:02)
```

> **Key point:** Same playlist, different traversal. The iterator pattern separates *how* you traverse a collection from the collection itself. Want repeat mode? Write a `RepeatIterator`. The playlist never changes.

---

## Comparison

| Pattern | Problem | Solution |
|---|---|---|
| **Factory Method** | Client code tied to specific concrete classes | A factory decides which class to instantiate based on input |
| **Facade** | Client must coordinate many subsystem calls | One class wraps the subsystems behind a simple interface |
| **Iterator** | Exposing internal structure to traverse a collection | Implement `__iter__`/`__next__` so `for` loops just work |

## Discussion Points
- How does Factory Method relate to OCP? (Adding a new codec means adding a new class and one dict entry — existing code stays untouched)
- When would you NOT use a Facade? (When clients genuinely need fine-grained control over subsystems — a facade can become a bottleneck if it hides too much)
- Why did we separate `ShuffleIterator` from `Playlist` instead of adding a `shuffle` flag? (SRP — the playlist stores songs, iterators define traversal. Mixing them means the playlist grows with every new traversal strategy)
- How do these three patterns connect in our UML diagram? (Factory creates codecs for the player, Facade hides the player's subsystems, Iterator lets us loop through playlists — each pattern solves a different structural problem in the same system)
- What UML relationship type is the Factory-to-Codec connection? (Dependency — the factory creates codecs but doesn't own them long-term)
