---
title: Extend View Controllers with Behavior Objects Right from Within Interface Builder
created_at: 2016-03-14 13:57:08 +0100
kind: worklog
tags: [ view-controller, behavior, composition, interface-builder ]
vgwort: http://vg01.met.vgwort.de/na/7339b612307b4f2b8e7258f64cd23458
comments: on
---

Krzysztof Zabłocki ([@merowing_](https://twitter.com/merowing_)) shared [his approach to "Behaviors"][obj] in a Slack channel the other day. These are functional extensions to view controllers that you can wire **via Interface Builder** (!) to inject behavior into a scene.

Think about how you would implement a parallax scrolling background. Or a login mechanism you use in 2+ places but with different design. Where'd you put the scrolling logic? How would you extract the login stuff?

The following paths are common:

1. **The naive alternative is to let the view controller do everything.** This results in massive view controller syndrome pretty quickly. It also isn't very reusable, so you'll end up with lots of copy & paste code when under time pressure. In the parallax example, the scene's view controller has 2 scroll views and changes the background relative to the foreground via delegate callbacks that tell the scrolling offset of the foreground.
2. **A little bit more sophisticated approach is to split a complex scene into multiple view controllers** and let each handle a particular component. One controller takes care of scrolling foreground and background while another displays the actual contents. For the login example, you could move the logic and component handling into a view controller into another Nib and import it from your main storyboard via view controller references.

Now the behavioral collaborators Krzysztof presents us are similar to the latter approach: you extract controller logic into different classes. Only they are no part of the view controller stack or view hierarchy. There's only the scene's controller and then there are behavior objects which are attached to it. Behaviors are plugged-in.

The difference is subtle: behaviors are plugged in and you tell them which views to watch and where to report changes. Apart from well-defined input and output ports, they are self-contained. Sub-view controllers would own the components while they react to their changes. They are the algorithms _plus_ everything that makes them view controllers: view lifecycle management, for example.

[Watch the video][vid] to see how it works. [Example code is on GitHub.](https://github.com/krzysztofzablocki/BehavioursExample) There are two examples I really like in the video:

1. Extracting image picker and display [at about 25:40](https://www.youtube.com/watch?v=QMVcIJz2sfg&feature=youtu.be&t=25m40s),
2. extracting parallax backgrounds from [about 23:38 onward](https://www.youtube.com/watch?v=QMVcIJz2sfg&feature=youtu.be&t=23m38s).

Scenes become very extensible this way instead of representing a preview or snapshot of the result. You can compose complex controller logic with behaviors without touching the view controller themselves much.

The image picker behavior [is defined like this](https://github.com/krzysztofzablocki/BehavioursExample/blob/master/BehaviourExample/Behaviours/ImagePicker/KZImagePickerBehaviour.h):

    #!objc
    //! obviously NS_OPTIONS would be better, but it's harder to expose that in XIB
    typedef NS_ENUM(NSUInteger, KZImagePickerBehaviourSourceType) {
      KZImagePickerBehaviourSourceTypeBoth = 0,
      KZImagePickerBehaviourSourceTypeCamera = 1,
      KZImagePickerBehaviourSourceTypeLibrary = 2,
    };

    //! Generates UIControlEventValueChanged when image is selected
    @interface KZImagePickerBehaviour : KZBehaviour
    //! source type to use
    @property(nonatomic, assign) IBInspectable NSInteger sourceType;

    //! image view to assign selected image to
    @property(nonatomic, weak) IBOutlet UIImageView *targetImageView;

    - (IBAction)pickImageFromButton:(UIButton *)sender;
    @end

A `KZBehaviour` exposes a weak `IBOutlet id owner` and uses `objc_setAssociatedObject` to attach the behavior to its owner (the view controller). That's all it provides. So the `KZImagePickerBehaviour` has a generic owner, a target image view, and an `IBAction` to start the process of picking an image. Simple.

[obj]: https://www.objc.io/issues/13-architecture/behaviors/
[vid]: http://youtu.be/QMVcIJz2sfg

Instead of coding each interaction paradigm into view controllers, add behaviors that do all of this on their own and are super reusable at the same time. This helps remedy massive view controller syndrom within the UI layer by extracting actual controller logic into different components.
