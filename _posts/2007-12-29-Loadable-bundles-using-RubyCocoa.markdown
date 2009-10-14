With the arrival of Leopard, Apple seems to be making a push to support Cocoa and Mac OS X development in languages other than Objective-C.  They ship both the PyObjC and RubyCocoa frameworks, and include several Xcode project templates for creating applications built upon these frameworks.  But what they forgot are the templates for building bundles that use these frameworks.

# Initializing RubyCocoa

If we take a look at the boilerplate we get from creating a new application using the "Cocoa-Ruby Application" template, we see the following in `main`:

    return RBApplicationMain("rb_main.rb", argc, argv);

This call performs any necessary initialization of the RubyCocoa bridge and then loads the specified Ruby script (rb_main.rb).  The Ruby script then sets up a few load paths and eventually ends with a call to `NSApplicationMain`.  This works quite well for an application, but we can't get away with this in a loadable bundle.  Thankfully the RubyCocoa authors are ahead of us, and have provided the `RBBundleInit` call.

`RBBundleInit` performs a similar function to `RBApplicationMain` in that it performs any necessary RubyCocoa initialization and then loads a specified Ruby script.  The parameters for this function are slightly different, so let's examine a call and break it down:

    RBBundleInit("ruby_bundle.rb", [MyRubyBundle class], nil);

The first parameter is obviously the Ruby script to execute.  The second parameter is a class, which RubyCocoa uses to find the bundle containing the specified Ruby script.  This parameter may be `nil`, in which case the first parameter would need to be an absolute path.  If the second parameter is not `nil`, then RubyCocoa searches for the bundle containing the class, and appends the bundle's resources path to the Ruby load path.  The third parameter is simply passed as an argument into the loaded script.


# Caveats

There are currently (as of 10.5.1) a couple of issues that unfortunately might discourage anyone from actually shipping a bundle written with RubyCocoa, depending on what application might be loading the bundle.

In the currently shipping versions of PyObjC and RubyCocoa, classes are created in the Objective-C runtime when either framework loads a bridge support file which contains inline functions.  Because of this, only one framework can safely be loaded into an application at any time.  [Bill Bumgarner][bbum] has written a great [example application][bbum1] which combines both PyObjC and RubyCocoa in a single application, and describes this problem in greater length.  

This means that writing bundles which are loaded into applications for which you as the developer do not have complete control is unpractical.  This includes System PreferencePanes, Automator Actions, Address Book plugins, and more.

The good news is that this has already been addressed in the trunk of the PyObjC Subversion repository.  And I hope that it will also be addressed in RubyCocoa the same way.  With any luck, these fixes will be included in the 10.5.2 update (anyone with the seeded version who can verify?).

# RubyCocoa templates

In anticipation of the aforementioned bug being fixed, I've started preparing some Xcode project templates for loadable bundle projects built with RubyCocoa.  At the time of this writing, templates exist for both a generic loadable RubyCocoa bundle and a RubyCocoa PreferencePane.  Eventually I may add a couple other templates such as Frameworks or Address Book plugins.

The templates can be found in my [Google Code repository][svn].

To install these templates, simply copy them to `~/Library/Application Support/Developer/3.0/Xcode`.


[bbum]: http://www.friday.com/bbum
[bbum1]: http://www.friday.com/bbum/2007/11/25/can-ruby-python-an-objective-c-co-exist-in-a-single-application/
[svn]: http://threeve.googlecode.com/svn/trunk/XcodeProjectTemplates/
