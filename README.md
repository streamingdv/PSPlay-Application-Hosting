# PSPlay Application Hosting

Welcome to the repository for PSPlay, a third-party Remote Play client available for Android, iOS (as MirrorPlay), and now for PCs with Windows and Linux OS. This repository serves as the hosting platform for the PSPlay application.

## Download
You can download the current version from the [Releases page](https://github.com/streamingdv/PSPlay-Application-Hosting/releases/).

PSPlay is currently supported on Android, iOS, Apple TV, Windows 10/11 and Linux x64 PCs.

## Key Features
- **Third-Party Controller Support**: Works with USB and Bluetooth controllers.
- **Bitrate Adjustments**: Manually adjust bitrate to optimize streaming quality.
- **Keyboard Support**: Offers basic keyboard functionality for navigation and gameplay.
- **Multiple Console Registration**: Allows registration of multiple consoles for easy switching.
- **Fullscreen Streaming Mode**: Fullscreen mode available with options to stretch and zoom without black bars.
- **Optimized User Interface**: User interface designed for easy navigation with both mouse and gamepad.
- **Advanced Decoding and Rendering**: Supports multiple decoder and rendering engines, including zero-copy rendering with image enhancement through Vulkan.
- **Optimized for Peak Streaming Performance**: Ensures a fast and stable streaming experience through extensive optimization and beta testing.

## Licensing
A valid license is required to use the full version of the PSPlay PC port and the standalone Android APK. Licenses can be [purchased here](https://streamingdv.com/shop-list-ns.html).
This is a one-time purchase that grants personal use across multiple devices. A single PC license key works for both Linux and Windows, while other platforms like Android require separate license keys.

## Android Version
Download the Android version on [Google Play](https://play.google.com/store/apps/details?id=psplay.grill.com). An alternative APK download is also available from the
latest [GitHub release](https://github.com/streamingdv/PSPlay-Application-Hosting/releases/). Please note that a valid Android APK license is required, which can be purchased [here](https://streamingdv.com/shop-list-ns).

## iOS / Apple TV
Download the iOS/Apple TV PSPlay port called MirrorPlay from the [App Store](https://apps.apple.com/us/app/mirrorplay-remote-streaming/id1638586503).

## Open Source Contributions
The development of PSPlay has been supported by several excellent open-source libraries:

- [JavaCV](https://github.com/bytedeco/javacv)
- [libplacebo](https://github.com/haasn/libplacebo)
- [OpenGLFX](https://github.com/husker-dev/openglfx)
- [LWJGL3](https://github.com/LWJGL/lwjgl3)
- [FFmpeg](https://github.com/FFmpeg/FFmpeg)

Parts of the PSPlay PC port, such as the Vulkan rendering pipeline and the gamepad handling via SDL2 through Jamepad, are open-sourced here:

- [libplacebo JNI project](https://github.com/streamingdv/libplacebo-jni)
- [Jamepad with D-Sense/ D-Shock features](https://github.com/grill2010/Jamepad)

## Contact
For any inquiries or issues with the application, feel free to reach out to me at [streamingdv@outlook.com](mailto:streamingdv@outlook.com).

Check out my other projects on GitHub or follow the development of PSPlay on [the PSPlay subreddit](https://www.reddit.com/r/PSPlay/).

**Disclaimer:** This project is not endorsed or certified by Sony Interactive Entertainment LLC.
