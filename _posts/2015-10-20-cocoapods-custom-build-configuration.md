---
title: Make Custom Debug Build Configurations Play Nicely With CocoaPods
created_at: 2015-10-20 13:10:29 +0200
kind: worklog
tags: [ cocoapods, xcode ]
comments: on
---

To test the Word Counter during development, I have long used a special build target. But that doesn't scale well if all I want is change a preprocessor macro to switch from file-based to in memory storage, for example.

I've never played around with build configurations in Xcode. They work well with Schemes, though, and setting up a build configuration and a scheme is much less overhead than maintaining a custom target.

{% include figure.md src="/assets/blog/2015/201510201316_add-config.png" alt="Duplicating config" caption="Hit the + button and then duplicate the existing Debug configuration." %}

From the project view, it's easy to duplicate the existing debug build configuration. 

I've done this twice for the Word Counter: once for an in memory store, once using test data. In the second case, the clock always points to the same time and the app uses different files. This way I don't destroy my personal records and have nice presets to make screenshots.

To be able to build and run the project using the new configuration, add a scheme.

{% include figure.md src="/assets/blog/2015/201510201321_add-scheme.png" alt="Scheme editor" caption="Duplicate the existing scheme using the cog icon" %}

From the scheme editor, duplicate the existing scheme for running the app and change its name. I added a "In Memory" suffix to have it read nicely in the list of schemes.

Change the used build configuration from the "Run" setting to the new configuration. You may also want to change the used configuration for testing, but I didn't, since the tests stay the same.

{% include figure.md src="/assets/blog/2015/201510201324_change-conf.png" alt="Change configuration" caption="Set the configuration to the one you've just created." %}

Now we have to tame CocoaPods.

## Configure CocoaPods

You will probably get a warning with `pod install`:

> [!] CocoaPods did not set the base configuration of your project because your project already has a custom config set. ...

In my case, there were a dozen of these.

They continued like this:

> In order for CocoaPods integration to work at all, please either set the base configurations of the target `InteractionTests` to `Pods/Target Support Files/Pods/Pods.debug in memory.xcconfig` or include the `Pods/Target Support Files/Pods/Pods.debug in memory.xcconfig` in your build configuration.

You see that the `InteractionTests` target under the "Debug In Memory" configuration is expected to use a different `.xcconfig`. Although CocoaPods created some new of these, I wasn't able to pick them in Xcode.

To solve the problem, I performed these steps:

1. Tell CocoaPods that the new configurations are descendants from the "Debug" configuration. I added `xcodeproj 'WordCounter', 'Debug In Memory' => :debug, 'Debug Test Data' => :debug` to my Podfile at root level.
2. Then I reset all newly created configurations. Instead of the `Pods.debug` configuration of some targets, I set all target configurations to "None".
3. Run `pod install` to make CocoaPods attach its `.xcconfig` files in the appropriate slots -- those I've just cleared.

Now I get different warnings: the test targets of the new configurations use the wrong `.xcconfig` files. Instead of `Pods-InteractionTests.debug.xcconfig`, it is set to `Pods.debug.xcconfig`.

Turns out this is because my 18-months-old podfile had global pods defined.

## Modern CocoaPods file

{% include figure.md src="/assets/blog/2015/201510201358_final-pods.png" alt="3 stages of pods" caption="First, reset all new configs to 'None'; then I found something doesn't seem to work. The third image shows what it's supposed to look like in the end." %}

Instead of:

    platform :osx, '10.9'
    use_frameworks!

    xcodeproj 'WordCounter', 'Debug In Memory' => :debug, 'Debug Test Data' => :debug

    link_with ['WordCounter', 'CedarSpecs', 'InteractionTests']

    pod 'CocoaLumberjack'
    pod 'LaunchAtLoginController', :git => "https://github.com/jashephe/LaunchAtLoginController.git"
    pod 'CorePlot', :git => 'https://github.com/core-plot/core-plot.git'

    target "CedarSpecs", :exclusive => true do
      pod 'Cedar'
    end

    target "InteractionTests", :exclusive => true do
      pod 'OCHamcrest', '~> 4.2'
      pod 'OCMockito', '~> 2.0' #OCHamcrest 4 support
    end

I now use:

    platform :osx, '10.9'
    use_frameworks!

    xcodeproj 'WordCounter', 'Debug In Memory' => :debug, 'Debug Test Data' => :debug

    def shared_pods
      pod 'CocoaLumberjack'
      pod 'LaunchAtLoginController', :git => "https://github.com/jashephe/LaunchAtLoginController.git"
      pod 'CorePlot', :git => 'https://github.com/core-plot/core-plot.git'
    end

    target "WordCounter" do
      shared_pods
    end

    target "CedarSpecs" do
      shared_pods
      pod 'Cedar'
    end

    target "InteractionTests" do
      shared_pods
      pod 'OCHamcrest', '~> 4.2'
      pod 'OCMockito', '~> 2.0' #OCHamcrest 4 support
    end

Seems to solves the exclusivity problem from before just fine. (Whatever that was. I don't remember anymore.)

No warnings left for me now and everything builds fine.

If that doesn't seem to work, delete the Pods folder from your project: mine contained stale and missing files. After deleting it, `pod install` added fresh files just fine.
