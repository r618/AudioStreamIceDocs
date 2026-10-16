# Changelog

## V 1.0 (06.2022)

- Internet/cloud/local media streaming
- iOS custom app controller

## V 1.1 (07.2022)

- added playlist items browsing support
- fixed local mp3 file tags
- fixed iOS platform Control Center + metadata
- also fixed a too tight init/download restriction

## V 1.2 (xx.2022)

- demo - added AudioTextures

## V 1.3 (09.2023)

- added an optional System.Net.HttpWebRequest backed `attemptToResolveUrlRedirection` flag to resolve Url redirects when connecting

## V 1.3.1 (12.2023)

- improved handling of the decoder thread when stopping/destroying
- MPEG decoder: fixed incorrect state for certain MPEG files
- MPEG decoder: updated NLayer

## 1.4.0

- updated dependencies/used libraries + added/replaced the existing MPEG decoding library
- cleaned up unneeded support files
- fixed loading of some (soma.fm) radio images from links in stream tags
- tags detected in streams are handled more efficiently - the notification/event was previously emitted even when their content didn't change
- the iOS plugin now uses `UnityBatchPlayerLoop()` to handle background audio mode by default, since it seems to be OK in the latest (~U6000.1) version(s)

### Demo application/scene

- the demo app/scene has a more responsive layout
- added a bunch of various online streaming formats/examples

Some servers don't produce a continuous stream of all content/tracks, but finish each track individually instead before starting a new one -
this is currently problematic for the Ogg/OPUS codec and is worked around in the demo scene such that the stream is restarted in the `OnPlaybackStopped` event after the previous track is finished.

## 1.4.1

- fixed incorrect url resolving for local media/files (don't use HttpHandler for links/urls not pointing to the network via "http")
- `Il2CppSetOptionAttribute` attributes used for IL2CPP builds are enabled by default - the necessary setup steps on how to use them properly were added to the initial import README doc
- submitted with Unity 2021.3.45f2

## 1.4.2

- fixed the default UnityWebRequest User-Agent not being accepted by some internet host(s)
- fixed a couple of bugs related to streaming, and unneeded allocations in several places
- submitted with Unity 2022.3.62

## 1.4.3

- compatible with the New Input System (no legacy Input Manager APIs are used by the package/demo, so it works with either input backend, or both)

## 1.4.4

### Added

- `validateServerCertificates` (default off = previous behaviour: any server certificate is accepted for connection and streaming; turn it on for the platform's default certificate validation)

### Demo

- request/server headers windows can be open at the same time
- updated audio textures

### iOS

- Command Center / headphones play-pause toggle works (missing `Platform_ToggledPlayPause` receiver)
- the platform (iOS) track image file is rewritten when the image changes (was written only once per station)

### Bugfixes/updates

- fixed metadata multibyte character encodings
- better Ogg/Vorbis track changes
- initial connections/redirects have better error recovery and don't block the main thread
- local ".opus" files are accepted as playlist entries

### Decoder/playback

- more performant, with better capacity estimates
