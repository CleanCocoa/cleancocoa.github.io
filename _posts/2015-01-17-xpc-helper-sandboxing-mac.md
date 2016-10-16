---
title: Getting XPC to a Helper App Working with Sandboxing Enabled
created_at: 2015-01-17 18:55:03 +0100
kind: worklog
tags: [ xpc, sandboxing, mac ]
vgwort: http://vg08.met.vgwort.de/na/1284005090f24e7ba41eb3a7ba690e0f
comments: on
---


Here's how I fixed the inter-process communication sample app called [AppSandboxLoginItemXPCDemo][demo] by Apple to make it work. When I tried it out at first, sandboxing ruined the day. You have to change a few things in order to make it work.

I'm trying to dip my toes into XPC for the Mac. The sample shows how to set-up a project in Xcode with a client app and a background service. It's very educational for my purpose. But it doesn't work out of the box.

**tl;dr**
:   You have to replace `$(TeamIdentifierPrefix)` in both `.entitlement` files with your real Apple developer team identifier. The placeholder won't be expanded to insert your ID automatically. (At least not during development. It is said to work when releasing for the App Store.)

The deal-breaker is App Sandboxing. If you turn it off, the communication between app and service work just fine. But you shouldn't turn it off. So how can we fix it? And how do we understand the problem?

First, replace committingrences of `XYZABC123.com.example` in the code with a real bundle identifier. In my case, this is `FRMDA3XRGC.de.christiantietze`, followed by `iDecide`, the demo app's name.

{% include figure.md src="/assets/blog/2015/201501171149_error.png" alt="Demo app error" caption="XPC error when you download AppSandboxLoginItemXPCDemo and try anything" %}

Let's start with what we see. Run the app, try to enter something, and let it fail. With Sandboxing enabled, this is the error you'll see in the app itself:

> Failed to query oracle: Error Domain=NSCocoaErrorDomain Code=4099 "Couldn’t communicate with a helper application." (The connection was invalidated.) UserInfo=0x600000077640 {NSDebugDescription=The connection was invalidated.}

Maybe it's a code signing issue. I updated the project settings as Xcode recommends to code sign the helper app. I even [checked code signing of both parts][helpersand] to see if the access right are mangled. Playing it safe, I set Gatekeeper to accept all kind of apps. The result of the checks:

    $ cd /Users/ct/Library/Developer/Xcode/DerivedData/iDecide-dmgouonuxnldtmfipbuctwsghqmm/Build/Products/Debug/

    $ spctl --assess -vvvv iDecide.app 
    iDecide.app: accepted
    override=security disabled
    origin=Mac Developer: Christian Tietze (933RH59P6T)

    $ spctl --assess -vvvv iDecide.app/Contents/Library/LoginItems/FRMDA3XRGC.de.christiantietze.iDecideHelper.app/
    iDecide.app/Contents/Library/LoginItems/FRMDA3XRGC.de.christiantietze.iDecideHelper.app/: accepted
    override=security disabled
    origin=Mac Developer: Christian Tietze (933RH59P6T)

Both are accepted. So there's nothing wrong it seems.

When you run the app, the Console has interesting things to say about this, though:

    kernel[0]: warning: debugserver(8322) performed out-of-band resume on iDecide(8321)
    lsboxd[198]: Not allowing process 8321 to launch "/Users/ct/Library/Developer/Xcode/DerivedData/iDecide-dmgouonuxnldtmfipbuctwsghqmm/Build/Products/Debug/iDecide.app" because it has not been launched previously by the user, 
    lsboxd[198]: Not allowing process 8321 to register app "/Users/ct/Library/Developer/Xcode/DerivedData/iDecide-dmgouonuxnldtmfipbuctwsghqmm/Build/Products/Debug/iDecide.app" for launch.
    appleeventsd[74]: <rdar://problem/11489077> A sandboxed application with pid 8321, "iDecide" checked in with appleeventsd, but its code signature could not be validated ( either because it was corrupt, or could not be read by appleeventsd ) and so it cannot receive AppleEvents targeted by name, bundle id, or signature. Error=ERROR: #100013  { "NSDescription"="SecCodeCopySigningInformation() returned 100013, -." }  (handleMessage()/appleEventsD.cp #2072) client-reqs-q
    sandboxd[1671]: ([8321]) iDecide(8321) deny mach-lookup FRMDA3XRGC.de.christiantietze.iDecideHelper

Lookup for the mach service our Launch Agent should get is denied, but we don't know why, yet.

To quit the helper service, I run `launchctl remove FRMDA3XRGC.de.christiantietze.iDecideHelper`, by the way.

A quick fix non-solution is to allow mach-lookup of a specific bundle identifier. Edit `iDecide.entitlements`, add the key `com.apple.security.temporary-exception.mach-lookup.global-name`, and make it accept an Array. Add an item with the name of the helper identifier, in my case `FRMDA3XRGC.de.christiantietze.iDecideHelper`. Run. It works.

So the XPC service is there, and it obtains a mach-service with the proper identifier, but usually, connections aren't allowed. Why is that?

With the mack-lookup exception in place, the Console warnings are these:

    kernel[0]: warning: debugserver(8740) performed out-of-band resume on iDecide(8739)
    appleeventsd[74]: <rdar://problem/11489077> A sandboxed application with pid 8739, "iDecide" checked in with appleeventsd, but its code signature could not be validated ( either because it was corrupt, or could not be read by appleeventsd ) and so it cannot receive AppleEvents targeted by name, bundle id, or signature. Error=ERROR: #100013  { "NSDescription"="SecCodeCopySigningInformation() returned 100013, -." }  (handleMessage()/appleEventsD.cp #2072) client-reqs-q

In other words, only the mach-lookup is denied. (And launching of the helper, according to the messages above, but `launchctl` shows it is indeed running.)

Meanwhile, I tried to change the code signing profiles to various combinations of things. Didn't work, but I encountered this message in the Console:

    xpcd[168]: restored permissions (100600 -> 100700) on /Users/ct/Library/Containers/FRMDA3XRGC.de.christiantietze.iDecideHelper/Container.plist

I didn't know about the container files. Taking a look at `Container.plist`, I find that the entitlements didn't seem to work properly. In this file, it says:

    SandboxProfileDataValidationEntitlementsKey = {
        com.apple.security.app-sandbox = :true;
        com.apple.security.application-groups = ( "de.christiantietze" );
    };

See that the team identifier is missing? The `.entitlements` files contain the value `$(TeamIdentifierPrefix)de.christiantietze`. `TeamIdentifierPrefix` is supposed to equal `FRMDA3XRGC`. The capabilities tab of the target in Xcode tells me that this will be the resulting value, and I trusted it.

But what if the variable is empty? What if I have to put my team identifier in there manually, who knows?

I try. Replace `$(TeamIdentifierPrefix)` with `FRMDA3XRGC.` and run the app.

It works.

Wait, what?!

I have no clue whose fault this is at the moment. I found hints that Xcode won't fill in `TeamIdentifierPrefix` when you use the development profile. If that's true, I've got an explanation at least.

But this leaves me startled: If you're not supposed to depend on the variable during development anyway, why use the team identifier for the helper's app and bundle name at all? Either you leave out the team identifier but are not allowed to distribute a sandboxed app, or you add it in the recommended way but can't test during development. That are be pretty poor choices.

This third option I have just discovered works, but it doesn't seem to be recommended. The "capabilities" tab of a target in Xcode suggests `$(TeamIdentifierPrefix)` to start the value of an `application-groups` item. All of this misbehaving leaves me with a feeling of having missed something in the process and now committing to a quick-fix.

You can't leave out the team identifier from the code, though. I've tried. Removing the team identifier in the app's lookup, when the `NSXPCConnection` is created, doesn't work. The app presents an error right away because the helper app can't even be found. Of course it can't: its bundle still contains the team identifier. So you're pretty much left with only one option: to add the team identifier manually.

This is weird, but at least it works.


<%= advertise :macmultiprocbook %>

[demo]: https://developer.apple.com/library/mac/samplecode/AppSandboxLoginItemXPCDemo/Introduction/Intro.html

[helpersand]: https://developer.apple.com/library/mac/documentation/Security/Conceptual/AppSandboxDesignGuide/AppSandboxInDepth/AppSandboxInDepth.html#//apple_ref/doc/uid/TP40011183-CH3-SW29
