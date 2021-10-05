---
layout: post
comments: true
title:  "How to run hello_xr on Oculus Quest"
date:   2021-04-06 13:46:00 +0930
tags: code xr android oculus quest
categories: code
---

This post describes how to get the OpenXR native sample application known as hello_xr running on Oculus Quest and Quest 2 hardware. 

As of writing there does not appear to be a comprehensive step-by-step guide on how to build hello_xr for Android and get it running on Oculus Quest hardware. There are a few dot points on the Oculus Mobile developer site, but it took me some time to get things working. Hopefully this saves the next person that time. 

Note that these instructions could well become obsolete as parts of this process change. Also note that these instructions are written for use on Windows but I don't imagine it will be too difficult to adapt this guide for other platforms.

# Prerequisite Steps

There are a few things we need to do before we begin in order to get the Oculus Quest hardware and Android Studio set up correctly

1. Follow the steps in [Oculus Mobile Device Setup] and enable developer mode on your device
2. Follow the steps in [Android Studio Setup for Oculus Mobile Development] to install Android Studio and set it up to build Oculus mobile applications

# Download the Oculus OpenXR Mobile SDK

As of writing, the standard OpenXR Android loader does not appear to work for Oculus Quest, so we have to link against the custom OpenXR loader library provided by Oculus. This is found inside the Oculus OpenXR Mobile SDK, which appears to contain a small number of additional resources for OpenXR native development on Oculus Quest hardware 

1. Download the [Oculus OpenXR Mobile SDK]
2. Extract the zip to a reasonable location, eg (C:\ovr_openxr_mobile_sdk_1.0.13)
3. Create an environment variable called OCULUS_OPENXR_MOBILE_SDK and set it to the path to this directory. This will help us configure the build system to use files from here. Do this using the Control Panel UI or use the following command line
{% highlight console %}
setx OCULUS_OPENXR_MOBILE_SDK C:\ovr_openxr_mobile_sdk_1.0.13
{% endhighlight %}

# Clone the OpenXR SDK Source Repository

There are two repositories on GitHub for the OpenXR SDK. We want the one called OpenXR-SDK-Source since it includes the hello_xr sample application

1. Clone or download OpenXR SDK Source from the [OpenXR SDK Source] GitHub repository to a reasonable location on your local machine

# Modify CMakeLists.txt file

Since by default OpenXR is configured to build a standard Android loader (which as of writing does not appear to work with Quest) we need make some local modifications to the build scripts

1. Within the OpenXR-SDK-Source directory, open the file \src\CMakeLists.txt
2. Replace the line 
{% highlight cmake %}option(BUILD_LOADER "Build loader" ON) {% endhighlight %} with 
{% highlight cmake %}option(BUILD_LOADER "Build loader" OFF) 
add_library(openxr_loader SHARED IMPORTED)
set_property(
        TARGET
        openxr_loader
        PROPERTY
        IMPORTED_LOCATION
        $ENV{OCULUS_OPENXR_MOBILE_SDK}/OpenXR/Libs/Android/${ANDROID_ABI}/${CMAKE_BUILD_TYPE}/libopenxr_loader.so
){% endhighlight %}

This tells CMake not to build the standard OpenXR Android loader, and to instead import the shared library provided by Oculus that we downloaded in an earlier step. We use the environment variable we set in that step to locate the directory, and we use CMake variables to select the right library based on the current build configuration

# Create an Android Studio Project

Create a new Android Studio Project. In the wizard:
1. Select the "No Activity" Project Template
2. Enter whatever project and package name you want
3. In Minimum SDK, select API 24

# Link the OpenXR SDK Source project with Gradle

1. Right click on the "app" module in the Android studio project and select "Link C++ Project with Gradle"
2. For the Build System, select CMake
3. For the Project Path, locate and select the CMakeLists.txt at the root level of the Open-SDK-Source repository that we cloned in the earlier step

# Modify build.gradle file

1. Open the build.gradle file for the "app" module. Within the Android Studio Project directory, it should be under app\build.gradle
2. Inside the android->defaultConfig block, add the block
{% highlight kotlin %}
externalNativeBuild {
	ndk {
		abiFilters 'arm64-v8a', 'armeabi-v7a'
	}
}
{% endhighlight %}

This tells the build system to only build for the two Application Binary Interfaces that we have loaders for

# Build the Application

At this point you should be able to build through Android Studio without any errors, though we still need to make some more changes before we can run the application on the device.

# Modify the AndroidManifest.xml

1. Replace the AndroidManifest.xml in the AndroidStudio project with the one provided in the OpenXR-SDK-Source/ repository.
- The one in the Android Studio project directory should be under app\src\main\AndroidManifest.xml, and the one in the Open-SDK-Source directory should be under OpenXR-SDK-Source\src\tests\hello_xr\AndroidManifest.xml
3. In the newly replaced AndroidManifest.xml file, change the block 
{% highlight xml %}<intent-filter>
	<action android:name="android.intent.action.MAIN" />
	<category android:name="android.intent.category.LAUNCHER" />
</intent-filter>{% endhighlight %}
to 
{% highlight xml %}<intent-filter>
	<action android:name="android.intent.action.MAIN" />
	<category android:name="com.oculus.intent.category.VR" />
	<category android:name="android.intent.category.LAUNCHER" />
</intent-filter>{% endhighlight %}

# Specify the Graphics Plugin

Before running, we have to specify the graphics plugin to use by running one of the following commands:

adb shell setprop debug.xr.graphicsPlugin OpenGLES  
adb shell setprop debug.xr.graphicsPlugin Vulkan

# Run the Application

Congratulations! You should now be able to run the hello_xr sample application on Quest and Quest 2 hardware. If everything worked, you should see a set of colorful cubes around the place as well as cubes attached to your controllers.

[Oculus Mobile Device Setup]: https://developer.oculus.com/documentation/native/android/mobile-device-setup/
[Android Studio Setup for Oculus Mobile Development]: https://developer.oculus.com/documentation/native/android/mobile-studio-setup-android/
[OpenXR SDK Source]: https://github.com/KhronosGroup/OpenXR-SDK-Source/
[Oculus OpenXR Mobile SDK]: https://developer.oculus.com/downloads/package/oculus-openxr-mobile-sdk/

{% if page.comments %}
  {% include disqus.html %}
{% endif %} 