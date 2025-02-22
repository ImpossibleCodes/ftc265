# ftc265

[![Discord](https://img.shields.io/discord/733961104807428140?color=%23738ADB&label=Join%20the%20Discord&logo=discord&logoColor=white)](https://discord.gg/85hZ4dnBUd)

ftc265 is a plug-and-play Intel RealSense T265 VSLAM camera wrapper for FTC based off my work in FRC
writing a [roboRIO T265 wrapper](https://github.com/Spartronics4915/SpartronicsLib).

## Installation
To get started you can either copy the
[example project](https://github.com/pietroglyph/FtcRobotController/tree/ftc265-example), or you can install manually.

### Manual Installation
Paste the following into `TeamCode/build.gradle`:

```gradle
repositories {
    // This line will already be here; you do not need to add it
    maven { url = "https://dl.bintray.com/first-tech-challenge/ftcsdk/" }
    // Add this line
    maven { url = "https://maven.0x778.tk" }
}
```

At the bottom of `TeamCode/build.gradle` add the following (if you have an existing `dependencies` block you can just
add the `implementation` line to that dependencies block):
```gradle
dependencies {
    // There should be a bunch of implementation configurations here already

    implementation 'com.spartronics4915.lib:ftc265:3.0.0'
}
```

Finally, set the `sourceCompatibility`, `targetCompatibility`, and `minSdkVersion` in the `build.common.gradle` at the
project root if you haven't already:
```gradle
android {
    // NOTE: There will be *way* more things in this file than what's shown here.
    // The blocks shown here should already exist, and you'll only need to change these three shown parameters.

    defaultConfig {
        minSdkVersion 24
    }
 
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}
```

You'll need to perform a Gradle sync in Android studio (if you use that IDE) after adding the new
repositories and dependencies.

## Usage
Basic usage is as follows:

```java
// Statically import the slamera object that starts automatically with the app.
import static com.spartronics4915.lib.T265StartupHook.slamera;

// This is the transformation between the center of the camera and the center of the robot
Pose2d cameraToRobot = new Pose2d();
// Increase this value to trust encoder odometry less when fusing encoder measurements with VSLAM
double encoderMeasurementCovariance = 0.8;
// Set to the starting pose of the robot
Pose2d startingPose = new Pose2d();

slamera.setOdometryInfo(cameraToRobot, encoderMeasurementCovariance);

slamera.setPose(startingPose); // Useful if your robot doesn't start at the field-relative origin

// Now we can grab our last received pose. The camera automatically starts with the app.
while (true) {
    slamera.getLastReceivedCameraUpdate();
}
```

Alternatively, you can also use the pre-made localizer by importing `com.spartronics4915.lib.T265Localizer` and
then adding this line to your drivetrain class in the constructor:

```java
setLocalizer(new T265Localizer());
```

There is also a ready-to-use example project
[here](https://github.com/pietroglyph/FtcRobotController/tree/ftc265-example).

Please note that the above example uses the simple synchronous API. There is also a more advanced
callback-based API available.

## Troubleshooting

### My camera doesn't connect
If you're getting a "No camera connected" exception then you can try a few things:
 1. Connect the camera to a computer running realsense-viewer and make sure that it works there.
 2. If you're using a USB hub, make sure to plug the hub in first, power the hub on (if applicable), and then connect
the camera.
 3. If you're using the Control Hub and your camera won't connect when you first boot up, try plugging the camera into
the USB 2 port on subsequent boots.
 4. If nothing works then you can open an issue or [join the Discord server](https://discord.gg/85hZ4dnBUd).

### The app hangs when I instantiate a `T265Camera`
This is likely happening because you haven't granted the correct permissions. The easiest way to
get a permissions prompt is to uninstall and reinstall the robot controller app. After reinstalling
you should get a permissions prompt on first run.

### I'm only getting estimated poses of (0, 0)
~~It's possible that you used the wrong overload of `start`. If you used the overload of the `start`
method that takes a `Consumer<CameraUpdate>` (a camera update callback) then you won't be able to
use `getLastRecievedCameraUpdate` unless you use the overload of `start` that takes no parameters.~~

This should no longer happen in v3 and up. Try asking in the Discord server.

## FAQs

### Is this legal?
According to [this GDC clarification](https://ftc-qa.firstinspires.org/qa/58), yes.

If that doesn't convince you, please take the following into account:
 1. In the opinion of the author, this is no more of a coprocessor than a servo. The T265 has an ASIC just like a servo
might have a microcontroller, but critically, neither of these are reprogrammable. Similar devices that do offboard
nonprogrammable video processing have been allowed (e.g. the Pixy cam, which was cited in the GDC clarification linked
above.)
 3. This fits under the broad category of UVC-compatible devices.

### Is the T265 accurate in an FTC context?
In my testing, yes. I used this in-season at (more demanding) FRC speeds, and when testing on an
FTC-field-sized area I consistently got less than 1 inch of error.

### Why use VSLAM?
 1. The T265 is smaller and easier to mount and maintain than sprung odometry wheels. It is also likely as accurate.
 2. Awards.

### Is this compatible with the REV Control Hub?
Yes. This should work out of the box on the Control Hub.

### Does this support importing and exporting relocalization maps?
Yes. It may be very buggy though, considering that you would need to call `slamera.stop()`, `slamera.free()`,
then re-construct the `slamera` object with `slamera = new T265Camera(...)`, where you can pass in your map.
From there, you can call `slamera.start()` to start the camera. Exporting maps is not hard, though you will
need to restart the camera after exporting.

### Why do you use RoadRunner geometry classes instead of library X's geometry classes?
Unfortunately, there is no canonical set of geometry classes (`Pose2d`, `Vector2d`, etc.) in FTC,
so I just had to pick a set to use. I originally picked the FTCLib geometry classes because they're
directly copied from WPILib (which I'm quite familiar with). [Ryleu](https://github.com/ryleu) ended
up changing this to use the RoadRunner classes for v3.0 and up for better compatibility with
RoadRunner (which many people use with this library). 

### How did you make this work?
I had to write JNI bindings to C++ librealsense because no Java bindings exist for the T265. Then I
had to patch librealsense to get around some bugs and Android incompatibilities (especially related
to the maximum size for USB bulk transfers and the list of USB VID/PID combos); this is why I
distribute a custom version of librealsense.

### Can I contribute?
Yes. Please report bugs or (even better) make a PR. FTC is not my main focus, and I do not currently
have access to testing hardware, so I would appreciate it if any nontrivial PRs are accompanied by a
test on real hardware (or even better, a unit test.)

### Will you support this into the future?
This is contingent on the community's response and Intel's actions. If ftc265 becomes popular then I
will continue to support it as best I can into the future until Intel provides a reasonable
alternative. Please note that I have limited time and that FTC is not my main focus; I will endeavor
to fix issues and respond to you as fast as possible, but expect the standard deviation on my
response time to be high :)

## License

Copyright 2021 Declan Freeman-Gleason

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU Lesser General Public License as
published by the Free Software Foundation, either version 3 of the
License, or (at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU Lesser General Public
License along with this program.  If not, see <https://www.gnu.org/licenses/>.
