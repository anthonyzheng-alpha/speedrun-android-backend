# Speedrun Android Backend

An interface for accessing Speedrun's Rust backend inside Speedrun Android. This
allows Speedrun Android to re-use the computer version's business logic and webpages,
instead of having to reimplement them.

This is a separate repo that gets published to a library that Speedrun Android consumes,
so that Speedrun Android development is possible without a Rust toolchain installed.

Credit to [Anki-Android-Backend](https://github.com/ankidroid/Anki-Android-Backend).

## Clone

Clone beside [speedrun-android](https://github.com/anthonyzheng-alpha/speedrun-android) with the exact folder name `speedrun-android-backend` (hard-coded in Gradle):

```bash
git clone --recurse-submodules https://github.com/anthonyzheng-alpha/speedrun-android-backend speedrun-android-backend
```

If you already cloned without submodules:

    git submodule update --init --recursive

Expected layout:

```
parent-folder/
  speedrun-android/
  speedrun-android-backend/   ← this repo
```

## Prerequisites

We assume you already have Android Studio, and are able to build the
[speedrun-android](https://github.com/anthonyzheng-alpha/speedrun-android) project.

The repos must sit in the same parent folder. Do not rename `speedrun-android-backend`.
Unless stated otherwise, all commands below are supposed to be executed in the current repo.

### Download Anki submodule

    git submodule update --init --recursive

### C toolchain

Install Xcode/Visual Studio if on macOS/Windows.

### Rust

Install rustup from <https://rustup.rs/>

#### macOS / Rust / Android Studio interaction

Note that Android Studio does a gradle sync when you open Speedrun-Android-Backend as a project,
and this sync requires the `cargo` command to be in your environment `PATH` variable to succeed.

If you get an exception related to `cargo` not found, you may need to alter your environment
before starting Android Studio. On macOS at least, you may open `Terminal`, verify `cargo` is
in the `PATH` with a `which cargo`, and open Android Studio directly with this environment setup
via the command `open -a "Android Studio"`

### Ninja

Speedrun can be built with Ninja or N2. N2 gives better status output and may be installed like so:

`./anki/tools/install-n2`

On Windows:

`bash anki/tools/install-n2`

*Note:* n2 receives occasional mandatory updates. If you see build errors, you may need to re-run this command and re-try the build

### NDK

#### Command-line install

Assuming `sdkmanager` from the "Command-Line Tools" package is in your PATH:

```bash
cargo install toml-cli
ANDROID_NDK_VERSION=$(toml get gradle/libs.versions.toml versions.ndk --raw)
sdkmanager --install "ndk;$ANDROID_NDK_VERSION"
```

#### GUI install

In Android Studio, choose the Tools>SDK Manager menu option.

- In SDK tools, enable "show package details"
- Choose NDK version listed in `gradle/libs.versions.tml` for the `ndk` key
- After downloading, you may need to restart Android Studio to get it to
synchronize gradle.

### Windows: msys2

Install [msys2](https://www.msys2.org/) into the default folder location.

After installation completes, run msys2, and run the following command:

```
pacman -S git rsync
```

When following the build steps below, make sure msys is on the path:

```
set PATH=%PATH%;c:\msys64\usr\bin
```

## Building

Two main files need to be built:

- The main .aar file, which contains the backend Kotlin code, web assets, and
Speedrun backend code compiled for Android.
- A .jar that contains the backend code compiled for the host platform, for use
with Robolectric unit tests.

You should do the first build with the provided shell .sh/.bat file, as it will
take care of downloading the target architecture library as well. You'll need
to tell the script to use the Java libraries and NDK downloaded by Android Studio,
eg on Linux:

```
cargo install toml-cli
ANDROID_NDK_VERSION=$(toml get gradle/libs.versions.toml versions.ndk --raw)
export ANDROID_HOME=$HOME/Android/Sdk
export ANDROID_NDK_HOME=$ANDROID_HOME/ndk/$ANDROID_NDK_VERSION
```

Or macOS:

```
cargo install toml-cli
ANDROID_NDK_VERSION=$(toml get gradle/libs.versions.toml versions.ndk --raw)
export ANDROID_HOME=$HOME/Library/Android/sdk
export ANDROID_NDK_HOME=$ANDROID_HOME/ndk/$ANDROID_NDK_VERSION
```

Or Windows using Powershell:

```
cargo install toml-cli
$env:ANDROID_NDK_VERSION=toml get gradle/libs.versions.toml versions.ndk --raw
$env:ANDROID_NDK_HOME="$env:ANDROID_HOME\ndk\$env:ANDROID_NDK_VERSION"
```

If you don't have Java installed, you may be able to use the version bundled
with Android Studio. Eg on macOS:

```
export JAVA_HOME="/Applications/Android Studio.app/Contents/jre/Contents/Home"
```

or Windows:

```
set JAVA_HOME=C:\Program Files\Android\Android Studio\jre
```

Now build with `./build.sh` or `build.bat`.

After you've confirmed building works, you may want to build again with the env
var RELEASE=1 defined, to build a faster version.

## Modify Speedrun Android to use built library

Now open the Speedrun Android project in AndroidStudio. To tell gradle to load the
compiled .aar and .jar files from disk, edit `local.properties`
in the Speedrun Android repo, and add the following line:

```
local_backend=true
```

Check `speedrun-android-backend/gradle.properties`'s `BACKEND_VERSION` and
`speedrun-android/build.gradle`'s `ext.ankidroid_backend_version`. Both variables
should have the same value. If it is not the case, you must edit Speedrun Android's
one.

After making the change, you should be able to build and run the project on an x86_64
emulator/device (arm64 on M1 Macs), and run unit tests.
