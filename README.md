# libspotify archive

This is an unofficial archive of the latest libspotify releases from Spotify,
as Spotify as of May 2018 no longer hosts these files themselves.


## WARNING: This library no longer works

In May 2015, libspotify was deprecated by Spotify and active maintenance
stopped. At this point, libspotify had been the main way to integrate with
Spotify for six years, and was part of numerous open source projects and
commercial applications, including many receivers and even cars.  It remained
the only API for playback outside Android and iOS.

In February 2016, server side changes to the Spotify API caused the search
functionality to stop working, without Spotify ever acknowledging it. Users
could work around this by using the Spotify web API for searches and
libspotify for playback.

In April 2022,
[Spotify announced](https://developer.spotify.com/community/news/2022/04/12/libspotify-sunset/)
that they would sunset the libspotify API one month later.

In May 2022, new libspotify connections to Spotify started failing.


## Downloads

These are the latest available releases for each CPU architecture, all released
around 2012.

| OS      | Architecture | Version                                                   | SHA256 checksum                                                    |
| ------- | ------------ | --------------------------------------------------------: | ------------------------------------------------------------------ |
| iOS     | ARM/i386     | [12.1.64](libspotify-12.1.64-iOS-universal.zip)           | `b32e9183e552c99bb4149e71181fadb26694553cab37a92311be16c286e0736a` |
| Android | ARM          | [12.1.51](libspotify-12.1.51-Android-arm-release.tar.gz)  | `754957de2648e7235e6ead323c22c111282adfc889535a2684c13067d2099505` |
| Win32   | x86          | [12.1.51](libspotify-12.1.51-win32-release.zip)           | `7c08475997461c077f79130d3cd1002111448c0ad321025748ffade7a37dda30` |
| macOS   | Universal    | [12.1.51](libspotify-12.1.51-Darwin-universal.zip)        | `80053f0779f6192a8052732904d88b91acc62a350831f6b585a3c6ac10cb8fbd` |
| Linux   | amd64        | [12.1.51](libspotify-12.1.51-Linux-x86_64-release.tar.gz) | `43a14e0732ba6ae30078fac105d0e2998d04d5f5c396a4968386bc4e22491058` |
| Linux   | armv5t       | [12.1.51](libspotify-12.1.51-Linux-armv5-release.tar.gz)  | `4d96efcb1423864683917f40fb4df481491250a76cb29be3a235b3732a64fefc` |
| Linux   | armv6t       | [12.1.51](libspotify-12.1.51-Linux-armv6-release.tar.gz)  | `4fb888eeb486578fa3a08e15f5aa2101632e60b56a068553d05d5d4ee0a080cc` |
| Linux   | armv6hf      | [12.1.103](libspotify-12.1.103-Linux-armv6-bcm2708hardfp-release.tar.gz) | `d658e6c1978fb46cf33376eb8367a51d024f4014f21beac1dd264532bcc54b24` |
| Linux   | armv7        | [12.1.51](libspotify-12.1.51-Linux-armv7-release.tar.gz)  | `ad27b6c5aee5382b66b39bfea3b1752076b7abcc445979ce25c1ec9d7ff3aeda` |
| Linux   | i686         | [12.1.51](libspotify-12.1.51-Linux-i686-release.tar.gz)   | `941ab4ba10bcd6ec4e96127afd095a39e11bc955de0882734c97e4f588b155ae` |


## Documentation

- API reference: See `share/doc/libspotify/html/index.html` in the Linux
  releases above.
- Examples: See `share/doc/libspotify/examples/` in the Linux releases above.
- [libspotify FAQ](faq.md)
