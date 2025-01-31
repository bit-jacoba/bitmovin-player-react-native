# Bitmovin Player React Native

Official React Native bindings for Bitmovin's mobile Player SDKs.

[![npm](https://img.shields.io/npm/v/bitmovin-player-react-native)](https://www.npmjs.com/package/bitmovin-player-react-native)
![Platforms](https://img.shields.io/badge/platforms-iOS%20%7C%20tvOS%20%7C%20Android%20%7C%20Android%20TV-lightgrey.svg)
[![MIT License](https://img.shields.io/badge/license-MIT-brightgreen.svg)](LICENSE)
[![Bitmovin Community](https://img.shields.io/discourse/users?label=community&server=https%3A%2F%2Fcommunity.bitmovin.com)](https://community.bitmovin.com/?utm_source=github&utm_medium=bitmovin-player-react-native&utm_campaign=dev-community)

> The library is under active development.

- [Bitmovin Player React Native](#bitmovin-player-react-native)
  - [Platform Support](#platform-support)
  - [Installation](#installation)
    - [Add package dependency](#add-package-dependency)
    - [Setup iOS Player SDK](#setup-ios-player-sdk)
    - [Setup Android Player SDK](#setup-android-player-sdk)
  - [Getting Started](#getting-started)
    - [Setting up a license key](#setting-up-a-license-key)
      - [Configuring through code](#configuring-through-code)
    - [Setting up a playback configurations](#setting-up-a-playback-configurations)
      - [Configuring `Info.plist`](#configuring-infoplist)
      - [Configuring `AndroidManifest.xml`](#configuring-androidmanifestxml)
    - [Accessing native `Player` instances](#accessing-native-player-instances)
    - [Listening to events](#listening-to-events)
    - [Enabling DRM protection](#enabling-drm-protection)
      - [Prepare hooks](#prepare-hooks)
    - [Adding external subtitle tracks](#adding-external-subtitle-tracks)
  - [Contributing](#contributing)

## Platform Support

This library requires at least React Native 0.64+ and React 17+ to work properly. The currently supported platforms are:

- iOS 12.0+
- tvOS 12.0+
- Android API 16+
- Android TV API 17+
- Fire TV (just make sure the Android API level is at least 17+)

Please note that browsers and other browser-like environments such as webOS and Tizen are not supported.

## Installation

Since Bitmovin's native SDKs are distributed through custom [Cocoapods](https://github.com/bitmovin/cocoapod-specs) and [Maven](https://artifacts.bitmovin.com/ui/native/public-releases) repositories, the installation cannot be managed by React Native's Autolink and requires some extra steps. Please refer to the installation instructions for each platform below. For more information on integrating the native SDKs, refer to the [Getting Started guides](https://bitmovin.com/docs/getting-started).

### Add package dependency

This library is available as an [NPM package](https://www.npmjs.com/package/bitmovin-player-react-native) and may be added as a dependency to your project using any node-based package manager, e.g.

> npm

```sh
npm install bitmovin-player-react-native --save
```

> yarn

```sh
yarn add bitmovin-player-react-native
```

### Setup iOS Player SDK

If you ran `pod install` after installing the node package and received an error similar to the one below, it is because Bitmovin's custom cocoapods repository has not been added to the `Podfile` and the [`iOS Player SDK`](https://github.com/bitmovin/bitmovin-player-ios-samples) could not be resolved:

```
[!] Unable to find a specification for `BitmovinPlayer (= 3.xx.x)` depended upon by `RNBitmovinPlayer`

You have either:
 * out-of-date source repos which you can update with `pod repo update` or with `pod install --repo-update`.
 * mistyped the name or version.
 * not added the source repo that hosts the Podspec to your Podfile.
```

To fix above error, open your `ios/Podfile` and set up Bitmovin's pods source url:

```ruby
require_relative '../node_modules/react-native/scripts/react_native_pods'
require_relative '../node_modules/@react-native-community/cli-platform-ios/native_modules'

# Bitmovin pods source url
source 'https://github.com/bitmovin/cocoapod-specs.git'

# iOS version should be 12 or greater.
# If you are running RN 0.69 you should be fine already.
platform :ios, '12.4'
install! 'cocoapods', :deterministic_uuids => false

target 'MyApp' do
  config = use_native_modules!

  # Rest of Podfile...
```

Now run `pod install` again (try with `--repo-update` if the error persists) - the error should now be resolved.

### Setup Android Player SDK

The Android setup also needs an extra step in order to correctly resolve the [Android Player SDK](https://github.com/bitmovin/bitmovin-player-android-samples) native dependency.

Just make sure to add Bitmovin's artifacts repository to the `allprojects.repositories` section of your `android/build.gradle`:

```groovy
allprojects {
    repositories {
        maven { url("$rootDir/../node_modules/react-native/android") }
        maven { url("$rootDir/../node_modules/jsc-android/dist") }
        mavenCentral {
            content {
                excludeGroup "com.facebook.react"
            }
        }
        google()
        maven { url 'https://www.jitpack.io' }
        // Add Bitmovin's artifacts repository url
        maven { url 'https://artifacts.bitmovin.com/artifactory/public-releases' }
    }
}
```

## Getting Started

The following is the simplest working component one can create using this library:

```typescript
import React, { useEffect, useCallback } from 'react';
import { View, Platform, StyleSheet } from 'react-native';
import {
  usePlayer,
  SourceType,
  PlayerView,
} from 'bitmovin-player-react-native';

export default function PlayerSample() {
  // The `usePlayer` hook creates or references a certain native `Player`
  // instance from within any component.
  const player = usePlayer({
    // The only required parameter is the license key but it can be omitted from code upon correct
    // Info.plist/AndroidManifest.xml configuration.
    //
    // Head to `Setting up a license key` for more information.
    licenseKey: '<ENTER-YOUR-LICENSE-KEY>',
  });

  useEffect(() => {
    // Load a streamable video source during component's initialization.
    player.load({
      // Select url and type dependeding on the running platform.
      url:
        Platform.OS === 'ios'
          ? // HLS for iOS
            'https://bitmovin-a.akamaihd.net/content/MI201109210084_1/m3u8s/f08e80da-bf1d-4e3d-8899-f0f6155f6efa.m3u8'
          : // Dash for Android
            'https://bitmovin-a.akamaihd.net/content/MI201109210084_1/mpds/f08e80da-bf1d-4e3d-8899-f0f6155f6efa.mpd',
      type: Platform.OS === 'ios' ? SourceType.HLS : SourceType.DASH,
      // Optionally set a title that will appear at player's top-left corner.
      title: 'Art of Motion',
      // Optionally load a poster image over the player.
      poster:
        'https://bitmovin-a.akamaihd.net/content/MI201109210084_1/poster.jpg',
      // Optionally set whether poster image will persist over player.
      // Useful for audio-only streams. Default to false.
      isPosterPersistent: false,
    });
  }, [player]);

  // onReady is called when the player has downloaded initial
  // video and audio and is ready to start playback.
  const onReady = useCallback(
    (event) => {
      // Start playback
      player.play();
      // Print event timestamp
      console.log(event.timestamp);
    },
    [player]
  );

  // Make sure to pass the `player` prop in `PlayerView`.
  return (
    <View style={styles.flex1}>
      <PlayerView style={styles.flex1} player={player} onReady={onReady} />
    </View>
  );
}

const styles = StyleSheet.create({
  flex1: {
    flex: 1,
  },
});
```

If you're interested in a complete running example, head to [`example/`](https://github.com/bitmovin/bitmovin-player-react-native/tree/main/example).

### Setting up a license key

First of all, create a license key on the [Dashboard](https://bitmovin.com/dashboard) and then make sure to associate your iOS app bundle id with it (see more [here](https://bitmovin.com/docs/player/getting-started/ios#step-3-configure-your-player-license)).

Then your license key can be either set from code or by configuring `Info.plist` and `AndroidManifest.xml`.

#### Configuring through code

```typescript
// Simply pass the `licenseKey` property to `PlayerConfig` when instantiating a player.

// With hooks
import { usePlayer } from 'bitmovin-player-react-native';
const player = usePlayer({
  licenseKey: '<ENTER-YOUR-LICENSE-KEY>',
});

// Without hooks
import { Player } from 'bitmovin-player-react-native';
const player = new Player({
  // Make sure to use React.createRef if instantiating inside a component.
  licenseKey: '<ENTER-YOUR-LICENSE-KEY>',
});
```

### Setting up a playback configurations

If needed, the default player behavior can be configured through the `playbackConfig` key when initialized.

```typescript
// Simply pass the `playbackConfig` property to `PlayerConfig` when instantiating a player.

// With hooks
import { usePlayer } from 'bitmovin-player-react-native';
const player = usePlayer({
  playbackConfig: {
    // Specifies whether the playback starts immediately after loading a source or not. Default is false.
    isAutoplayEnabled: true,
    // Specifies if playback starts muted. Default is false.
    isMuted: true,
    // Specifies if time shift for live streams should be enabled. Default is true.
    isTimeShiftEnabled: true,
    // Whether background playback is enabled or not. Default is false.
    // Only available for iOS.
    isBackgroundPlaybackEnabled: true,
  },
});

// Without hooks
import { Player } from 'bitmovin-player-react-native';
const player = new Player({
  // Make sure to use React.createRef if instantiating inside a component.
  playbackConfig: {
    isAutoplayEnabled: true,
    isMuted: true,
    isTimeShiftEnabled: true,
    isBackgroundPlaybackEnabled: true,
  },
});
```

#### Configuring `Info.plist`

Add the following lines to the `<dict>` section of your `ios/Info.plist`:

```xml
<key>BitmovinPlayerLicenseKey</key>
<string>ENTER-YOUR-LICENSE-KEY</string>
```

#### Configuring `AndroidManifest.xml`

Add the following line to the `<application>` section of your `android/app/src/main/AndroidManifest.xml`:

```xml
<meta-data android:name="BITMOVIN_PLAYER_LICENSE_KEY" android:value="ENTER-YOUR-LICENSE-KEY" />
```

### Accessing native `Player` instances

When you instantiate a player with `usePlayer` or `new Player()` from javascript, you're actually either creating a new `Player` instance in the native side (see [SDKs docs](https://bitmovin.com/docs/player/sdks) for more info) or referencing an existing one.

So it means that a player with the same `nativeId` in two different parts of the code is referencing the same in-memory instance internally.

**Example**

Both components in the example below are referencing the same native `Player` indexed as `my-player`. And even though each `<PlayerView />` creates a different `View` internally, the `Player` instance (which is a separate thing) remains the same. It just gets attached to a different view.

```typescript
// Using `usePlayer`
export const CompA = () => {
  // Same `player` as in `CompB`.
  const player = usePlayer({
    nativeId: 'my-player',
  });
  return <PlayerView player={player} />;
};

// Using `new Player()`
export const CompB = () => {
  // Same `player` as in `CompA`.
  const player = useRef(
    new Player({
      nativeId: 'my-player',
    })
  );
  return <PlayerView player={player.current} />;
};
```

### Listening to events

Both player and source events can be registered from `PlayerView`, but not all of them. For a complete list of the events currently available, checkout [`EventProps`](https://github.com/bitmovin/bitmovin-player-react-native/blob/main/src/components/PlayerView/events.ts#L29) and [`events.ts`](https://github.com/bitmovin/bitmovin-player-react-native/blob/main/src/events.ts).

To register an event callback, just pass its name prefixed with `on` as a `PlayerView` prop:

```typescript
return (
  <PlayerView
    onReady={onReady}
    onMuted={onMuted}
    onPaused={onPaused}
    onPlayerActive={onPlayerActive}
    onSourceLoaded={onSourceLoaded}
    onPlayerError={onPlayerError}
    onSourceError={onSourceError}
    onPlaybackFinished={onPlaybackFinished}
    {...}
  />
);
```

### Enabling DRM protection

> ⚠️ **Beta Version**: For now, only `FairPlay` is supported on iOS and
> only `Widevine` is supported on Android. More DRM systems will be added in the future.

Simple streaming of protected assets can be enabled with just a little configuration on `SourceConfig.drmConfig`:

```typescript
import { Platform } from 'react-native';
import { SourceConfig, SourceType } from 'bitmovin-player-react-native';

// Source configuration for protected assets.
const drmSource: SourceConfig = {
  // Protected stream URL.
  url:
    Platform.OS === 'ios'
      ? 'https://fps.ezdrm.com/demo/video/ezdrm.m3u8' // iOS stream url
      : 'https://bitmovin-a.akamaihd.net/content/art-of-motion_drm/mpds/11331.mpd', // Android stream url
  // Stream type.
  type: Platform.OS === 'ios' ? SourceType.HLS : SourceType.DASH,
  // DRM setup.
  // Each key in this object maps to a different DRM system config (`widevine` or `fairplay`).
  drmConfig: {
    // Widevine is the default and only DRM system supported on Android for now.
    widevine: {
      licenseUrl: 'https://cwip-shaka-proxy.appspot.com/no_auth',
    },
    // FairPlay is the default and only DRM system supported on iOS for now.
    fairplay: {
      licenseUrl:
        'https://fps.ezdrm.com/api/licenses/09cc0377-6dd4-40cb-b09d-b582236e70fe',
      certificateUrl: 'https://fps.ezdrm.com/demo/video/eleisure.cer',
    },
  },
};
```

#### Prepare hooks

In the native SDKs, some DRM properties like `message` and `license` can have their value transformed before use in order
to enable some more complex use cases: such as extracting the `license` from a `JSON`, for example.

In order to handle such transformations, it's possible to hook methods onto `SourceConfig.drmConfig` to proxy DRM values
and potentially alter them:

```typescript
import { Platform } from 'react-native';
import { SourceConfig, SourceType } from 'bitmovin-player-react-native';

// Source configuration for protected assets.
const drmSource: SourceConfig = {
  // Protected stream URL.
  url:
    Platform.OS === 'ios'
      ? 'https://fps.ezdrm.com/demo/video/ezdrm.m3u8' // iOS stream url
      : 'https://bitmovin-a.akamaihd.net/content/art-of-motion_drm/mpds/11331.mpd', // Android stream url
  // Stream type.
  type: Platform.OS === 'ios' ? SourceType.HLS : SourceType.DASH,
  // DRM setup.
  drmConfig: {
    // Widevine is the default and only DRM system supported on Android for now.
    widevine: {
      licenseUrl: 'https://cwip-shaka-proxy.appspot.com/no_auth',
      // Data is passed as a base64 string and expects to return a base64 string.
      prepareLicense: (license: string) => {
        // Do something with the `license` value...
        // And return processed data as base64 string.
        return license; // base64 string
      },
    },
    // FairPlay is the default and only DRM system supported on iOS for now.
    fairplay: {
      licenseUrl:
        'https://fps.ezdrm.com/api/licenses/09cc0377-6dd4-40cb-b09d-b582236e70fe',
      certificateUrl: 'https://fps.ezdrm.com/demo/video/eleisure.cer',
      // Data is passed as a base64 string and expects to return a base64 string.
      prepareLicense: (license: string) => {
        // Do something with the `license` value...
        // And return processed data as base64 string.
        return license; // base64 string
      },
      // Data is passed as a base64 string and expects to return a base64 string.
      prepareMessage: (message: string, assetId: string) => {
        // Do something with the `assetId` and `message` values...
        // And return processed data as base64 string.
        return message; // base64 string
      },
    },
  },
};
```

The [`FairplayConfig`](https://github.com/bitmovin/bitmovin-player-react-native/blob/development/src/drm.ts#L10) interface provides a bunch of hooks that can be used to fetch and transform different DRM related data. Check out the [docs](https://github.com/bitmovin/bitmovin-player-react-native/blob/development/src/drm.ts#L10) for a complete list and detailed information on them.

Also, don't forget to check out the [example](https://github.com/bitmovin/bitmovin-player-react-native/tree/development/example) app for a complete iOS/Android [DRM example](https://github.com/bitmovin/bitmovin-player-react-native/blob/development/example/src/screens/BasicDRMPlayback.tsx).

### Adding external subtitle tracks

Usually, subtitle tracks are provided in the manifest of your content (see [Enconding Manifests API](https://bitmovin.com/docs/encoding/api-reference/sections/manifests) for more information). And if they are provided this way, the player already recognizes them and show them in the subtitles selection menu without any further configuration.

Otherwise, it's also possible to add external tracks via the subtitle API:

```typescript
import { Platform } from 'react-native';
import {
  SourceConfig,
  SourceType,
  SubtitleFormat,
} from 'bitmovin-player-react-native';

// Source config with an external subtitle track.
const config: SourceConfig = {
  url:
    Platform.OS === 'ios'
      ? 'https://bitmovin-a.akamaihd.net/content/sintel/hls/playlist.m3u8'
      : 'https://bitmovin-a.akamaihd.net/content/sintel/sintel.mpd',
  type: Platform.OS === 'ios' ? SourceType.HLS : SourceType.DASH,
  poster: 'https://bitmovin-a.akamaihd.net/content/sintel/poster.png',
  // External subtitle tracks list to be added to this source.
  subtitleTracks: [
    // You can select 'Custom English' in the subtitles menu.
    {
      // The URL of the subtitle file. Required.
      url: 'https://bitdash-a.akamaihd.net/content/sintel/subtitles/subtitles_en.vtt',
      // External file format.
      // Supports `.vtt`, `.ttml` and `.cea` extensions.
      //
      // This option can be left empty since the player automatically recognizes the format
      // from the provided url most of the time.
      format: SubtitleFormat.VTT,
      // Label for this track shown under the selection menu. Required.
      label: 'Custom English',
      // The IETF BCP 47 language tag associated with this track. Required.
      language: 'en',
      // The unique identifier used for this track.
      // The default value for this options is a randomly generated UUID.
      identifier: 'sub1',
      // This track is considered the default if set to `true`.
      // The default value for this option is `false`.
      isDefault: false,
      // If set to `true` it means that the player should automatically select and switch this
      // subtitle according to the selected audio language. Forced subtitles do not appear in
      // `Player.getAvailableSubtitles`.
      //
      // The default value for this option is `false`.
      isForced: false,
    },
    // You may add even more tracks to the list...
  ],
};
```

The supported `PlayerView` events for subtitles are:

- `onSubtitleAdded`
- `onSubtitleRemoved`
- `onSubtitleChanged`

You might check out a complete subtitle example in the [`example/`](https://github.com/bitmovin/bitmovin-player-react-native/tree/development/example) app.

## Contributing

See the [contributing guide](CONTRIBUTING.md) to learn how to contribute to the repository and the development workflow.
