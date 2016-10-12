# Gradle plugin crash with `externalNativeBuild`

Version 2.2 of the Android Gradle Plugin added the ability to define an [external NDK build][nb].

Unfortunately, this crashes with a `NullPointerException` if a `local.properties` file exists that does not contain a `ndk.dir` property — even if a valid NDK has been defined via the standard `ANDROID_NDK_HOME` environment variable.

## Environment
Using the current latest tools:

* Tools: 25.2.2
* Build Tools: 24.0.3
* Platform Tools: 24.0.3

## Current behaviour
With this repository checked out, the following behaviours can be observed.

### Without NDK defined
As expected, the build fails:
```
$ ANDROID_NDK_HOME= ./gradlew clean assembleDebug
FAILURE: Build failed with an exception.

* What went wrong:
A problem occurred configuring root project 'repro'.
> NDK not configured.
...

BUILD FAILED
```

### With `ANDROID_NDK_HOME` defined
As expected, the build succeeds:
```
$ ANDROID_NDK_HOME=/path/to/android/ndk ./gradlew clean assembleDebug
:externalNativeBuildCleanDebug
:externalNativeBuildCleanRelease
:clean
...
:externalNativeBuildDebug
...
:assembleDebug

BUILD SUCCESSFUL
```

### With `ANDROID_NDK_HOME` defined, and a `local.properties` file
As no `ndk.dir` property has been defined in the `local.properties` file, we expect that the valid `ANDROID_NDK_HOME` value will be used.

However:
```
$ touch local.properties
$ ANDROID_NDK_HOME=/path/to/android/ndk ./gradlew clean assembleDebug
FAILURE: Build failed with an exception.

* What went wrong:
A problem occurred configuring root project 'repro'.
> java.lang.NullPointerException (no error message)
...

BUILD FAILED
```

## Desired behaviour
If a `local.properties` file exists, but does not define an `ndk.dir` property, then the external NDK build task should fall back to the `ANDROID_NDK_HOME` environment variable, in the same way that the existing `android.ndkDirectory` value in the Gradle plugin does.

[nb]:https://google.github.io/android-gradle-dsl/2.2/com.android.build.gradle.BaseExtension.html#com.android.build.gradle.BaseExtension:externalNativeBuild(org.gradle.api.Action)
