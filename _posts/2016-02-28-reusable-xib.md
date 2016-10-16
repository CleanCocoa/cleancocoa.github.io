---
title: Reusable View Components from Nibs
created_at: 2016-02-28 11:00:27 +0100
kind: worklog
tags: [ xcode, interface-builder ]
url: http://merowing.info/2012/12/quick-tip-for-interface-builder/
preview: fulltext
comments: on
---

Just found [this today](http://merowing.info/2012/12/quick-tip-for-interface-builder/) in a Slack channel. It seems we can actually reference and re-use Xib files by overriding `awakeFromCoder`!

> If we are loading from placeholder view, we create a real view and then we transfer some common properties from it and all it’s subviews, then we just return that instance in place of placeholder, otherwise we just return normal view (This method is implemented on NSObject so we can call super, but this still should be done with method swizzling instead of category smashing).

There's [a sample project on GitHub](https://github.com/krzysztofzablocki/XibReferencing/). Let's tear it apart.

It works like this:

    #!objc
    const int kNibReferencingTag = 616;
    
    @implementation UIView (NibLoading)
    // ...
    - (id)awakeAfterUsingCoder:(NSCoder *)aDecoder
    {
      if (self.tag == kNibReferencingTag) {
        //! placeholder
        UIView *realView = [[self class] loadInstanceFromNib];
        realView.frame = self.frame;
        realView.alpha = self.alpha;
        realView.backgroundColor = self.backgroundColor;
        realView.autoresizingMask = self.autoresizingMask;
        realView.autoresizesSubviews = self.autoresizesSubviews;
    
        for (UIView *view in self.subviews) {
          [realView addSubview:view];
        }
        return realView;
      }
      return [super awakeAfterUsingCoder:aDecoder];
    }

When the `kNibReferencingTag` is set, the current instance (`self`) is treated as a prototype. From that prototype we transfer common properties to the `realView` which is properly loaded from a Nib. That means it doesn't go the `kNibReferencingTag` path but the usual path, deferring to `super`.

The sample app contains `SomeView` with its own Xib that should be reused.

In [the app's Xib](https://github.com/krzysztofzablocki/XibReferencing/blob/master/XibReferencing/en.lproj/MainStoryboard.storyboard), we have a scene that uses it as follows:

    #!xml
    <scenes>
        <!--View Controller-->
        <scene sceneID="4">
            <objects>
                <viewController id="2" customClass="ViewController" sceneMemberID="viewController">
                    <view key="view" contentMode="scaleToFill" id="5">
                        <rect key="frame" x="0.0" y="20" width="768" height="1004"/>
                        <autoresizingMask key="autoresizingMask" widthSizable="YES" heightSizable="YES"/>
                        <subviews>
                            <view tag="616" contentMode="scaleToFill" id="VTu-Cz-9kE" customClass="SomeView">
                                <rect key="frame" x="113" y="216" width="100" height="100"/>
                                <autoresizingMask key="autoresizingMask" widthSizable="YES" heightSizable="YES"/>
                                <color key="backgroundColor" white="1" alpha="1" colorSpace="custom" customColorSpace="calibratedWhite"/>
                            </view>
                            <view tag="616" contentMode="scaleToFill" id="7te-Q6-moz" customClass="SomeView">
                                <rect key="frame" x="199" y="446" width="100" height="100"/>
                                <autoresizingMask key="autoresizingMask" widthSizable="YES" heightSizable="YES"/>
                                <color key="backgroundColor" white="1" alpha="1" colorSpace="custom" customColorSpace="calibratedWhite"/>
                            </view>
                            <view tag="616" contentMode="scaleToFill" id="7vx-xh-xRD" customClass="SomeView">
                                <rect key="frame" x="375" y="231" width="100" height="100"/>
                                <autoresizingMask key="autoresizingMask" widthSizable="YES" heightSizable="YES"/>
                                <color key="backgroundColor" white="1" alpha="1" colorSpace="custom" customColorSpace="calibratedWhite"/>
                            </view>
                        </subviews>
                        <color key="backgroundColor" white="1" alpha="1" colorSpace="custom" customColorSpace="calibratedWhite"/>
                    </view>
                </viewController>
                <placeholder placeholderIdentifier="IBFirstResponder" id="3" sceneMemberID="firstResponder"/>
            </objects>
        </scene>
    </scenes>

I know, Xib files aren't the best to read. Here's what it boils down to:

* Create a scene using a view controller of type `ViewController`. 
* In its main view place 3 subviews ...
    * of type `SomeView`,
    * all with the tag `616`,
    * and a few standard color properties -- apparently all set to white.  It doesn't matter anyway since the `SomeView.xib` will dictate what it really looks like.

Unlike `@IBDesignable` components, you won't have any live preview in Interface Builder. Just empty placeholder boxes.
