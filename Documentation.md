::o9.2o23::

# AudioStreamIce

## Basic usage

- add the `AudioStreamIce` component to a GameObject in the scene
- hit Play -> by default it should automatically start streaming the SpaceStation station of SomaFM (\*\*)
- check and customise the remaining fields - each one has its own tooltip with help/explanation

(\*\*) For HTTP (non-secure) links in Unity 2022.1 and up, please also set 'Allow downloads over HTTP' to 'Always allowed' in Player Settings

- the demo scene displays a warning if this is not enabled

## Basic scripting

- either assign a source to `.url` and call `.Play()`,
- or call `.Play(url)` - this won't modify `.url` on the component (also used in the demo to play an individual playlist entry while the original playlist location is not modified)
- the final url which will actually be played is stored in the public `.url_final` once resolved
- (optionally set the format if it's known)
- call `.PauseUnpause(bool pause)` and `.Stop()` to drive streaming/playback
- `OnPlaybackStarted`, `OnPlaybackPaused`, `OnPlaybackStopped`, `OnTagChanged` and `OnError` Unity events are called appropriately (\*)
- on iOS, the system is notified when tags change, and TITLE/ARTIST is set in the Command Center as the now playing item. For some stations, such as SomaFM, I also implemented the album art image (see [Tags](#tags) below).
  The appropriate Unity event is also called when the user interacts with the Play/Pause control in the Command Center (there are no other active controls there).

**NOTE:**

- all calls are delayed by one frame
- `.Play()`/`.Play(url)` is successfully executed only if the component is in the stopped state (`IsRunning` is false) one frame later
- `.PauseUnpause()` is successfully executed only if the component is streaming or paused (`IsPlayingOrPaused` is true) one frame later

(\*)

- you can call other components' methods from event handlers

Please see the demo scene for a complete example of all of the above.

## UnityWebRequest / System.Net.HttpWebRequest

- UnityWebRequest is used for all communication, except when attempting to resolve a Url redirect for the initial connection - for that, the platform's System.Net.HttpWebRequest is used. This is the default behaviour, which can be turned off via `attemptToResolveUrlRedirection`
- UnityWebRequest and System.Net.HttpWebRequest should both follow the system proxy settings
- by default any server TLS certificate is accepted when connecting and streaming (so self-signed/expired certificates of radio servers don't prevent playback) - set `validateServerCertificates` to use the platform's default certificate validation instead
- call `SetCustomRequestHeader(string key, string value)` to set up any custom request headers before calling `.Play()`. Those will be used for the initial connection/(playlist) and for all streaming.
- for 2022 and up, please enable non-secure HTTP downloads in Player Settings in order to use HTTP links in the demo:
  go to Edit -> Project Settings -> Player -> Other Settings -> Configuration and set 'Allow downloads over HTTP' to 'Always allowed'. Alternatively, use secure (HTTPS) links only.

## Supported formats

- decoding support is provided via OSS libraries (see LICENSE/COPYING for each in the `Libs` folder)
  - for MPEG streams:
    - MPEG 1 & 2 decoder for Layers 1, 2 & 3 via [MP3Sharp](https://github.com/r618/MP3Sharp) + [TagLib#](https://github.com/mono/taglib-sharp)
  - for Ogg streams:
    - Ogg/Vorbis via [NVorbis](https://github.com/NVorbis/NVorbis)
    - Ogg/OPUS via [Concentus](https://github.com/lostromb/concentus) + [Concentus.Oggfile](https://github.com/lostromb/concentus.oggfile)

  also with a few patches already merged upstream.

- if the selected format is AUTODETECT (default) on `.Play()`, the asset tries to play MPEG first, then Ogg/Vorbis and finally Ogg/OPUS format,
  except when the url ends with 'ogg' or 'opus', in which case it tries to play Vorbis, then OPUS and finally MPEG format.

## Playlists

- if the filename/url ends with one of the PLS, M3U or M3U8 extensions, it is treated as a playlist, which is downloaded/read first
  - based on the 'Retrieve All Playlist Items' checkbox, either
    - the first entry from the playlist is played automatically (default | OFF)
    - or, if it's enabled, user code is notified via an event of all found/supported playlist items and the operation stops
- the asset parses PLS/M3U/8 playlists according to [PLS](https://en.wikipedia.org/wiki/PLS_%28file_format%29), resp. [M3U](https://en.wikipedia.org/wiki/M3U)
  - PLS:
    - 'http/s' entries are returned as-is
  - M3U/8:
    - if a playlist entry is an existing directory on the local filesystem, the supported files of this directory are returned, non-recursively (nested directories are ignored)
  - supported/valid entries (in both formats) are either any 'http/s' links, or (local) files with ".mp3", ".ogg", ".opus", ".m3u", ".m3u8" or ".pls" extensions - other local entries are skipped
- playlists can contain other playlist/s (of either type)
- entries can be specified with relative paths (relative to the application/editor current working directory); paths are resolved via .NET/UnityWebRequest

Please see the demo scene, where it's possible to e.g. play each individual playlist entry when 'Retrieve All Playlist Items' is enabled.

## Tags

- tag changes are reported via Unity event/s as they arrive from the SHOUTcast/Icecast stream, and from files.
  All received 'Tag:Value' pairs are reported; if a binary picture tag contains a supposed image, a Texture2D is created based on it (and displayed in the demo).
  If a station publishes StreamUrl in its metadata and it contains a valid link to an image, the image is retrieved and used as the track image (this is e.g. the case for SomaFM).
- the ARTIST, TITLE and APIC_ (album art) tags + the above-mentioned image are used for the now playing info on iOS.

## Pausing the playback

Pausing the audio currently just silences it (sets the volume of the AudioSource to 0) while the download and streaming updates continue.

## Streaming and playback buffers

- the download buffer keeps downloaded data until it grows over ~4MB, after which already processed data is discarded (data not yet decoded is always kept, so a fully downloaded local file can be held in memory as a whole)
- additionally, up to the first ~2MB of the stream are kept for reading MPEG (ID3) tags, such as album art
- decoded audio is played from a circular PCM buffer set to a fixed length of ~15 seconds; if there's enough decoded data in the PCM buffer, the decoder is set to sleep until it's needed again
  (decoder activity is indicated by an asterisk next to the playback buffer in the demo scene)

## iOS

- on iOS, for background audio to work, please enable Custom Background Behaviours in the Player Settings:
  - Audio, AirPlay, PiP
- the rest of the setup is already implemented via a custom app controller and happens automatically
  (the iOS build needs
  - `Plugins/iOS/AudioStreamIceAppController.h`
  - `Plugins/iOS/AudioStreamIceAppController.mm`)

  [for a discussion about the used AVAudioSession see https://developer.apple.com/library/archive/qa/qa1668/_index.html#//apple_ref/doc/uid/DTS40010209]
- if the build fails due to a missing link step, please add MediaPlayer.framework manually
  (this is set via the Inspector in 202x and later; 2019 might be missing it):
  - in 2020 and later: in the Inspector Import Settings for `Plugins/iOS/AudioStreamIceAppController.mm`, enable MediaPlayer in 'Rarely used frameworks'
  - in 2019: add MediaPlayer.framework in Xcode to the Link Build step for the UnityFramework target
- for user interaction events in the iOS Command Center to work, the game object in the scene with the AudioStreamIce component has to be named 'AudioStreamIce' - this is because UnitySendMessage calls need a game object name to send the events to

> **Note:** the app controller supports OOTB only newer Unity (~2021) Xcode projects.
> It's necessary to fix its code (i.e. set the audio session) manually when building with earlier (~2019) Unity versions.

## Android

- Android is much more fragmented and, compared to iOS, there's no unified way of displaying e.g. media controls for all currently existing widely used Android versions, so this is not supported as of now.
- to keep an activity/application running when the user is not interacting with it, i.e. on the lock screen or in the background,
  various OS versions provide various ways to configure this for a concrete application - please refer to your Android version on how to do this.
- to keep the Unity player main activity running at all times, please
  - export the project when doing a build
  - in the resulting Gradle project, modify `UnityPlayerActivity.java`:
    - prevent the Unity player from pausing by e.g. commenting out `mUnityPlayer.pause();` in the `onPause()` activity method
  - the Unity player will now not stop its update loop when it otherwise would - combined with the above, this is enough for the asset to perform even when the activity is not active.

---

That's it, thank you for using AudioStreamIce!
