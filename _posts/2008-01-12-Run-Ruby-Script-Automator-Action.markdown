---
title: Run Ruby Script Automator Action
tags: ['Automator', 'Cocoa', 'MacOSX', 'Ruby']
layout: post
summary: >
    In a [recent blog post][RunPythonScript], [Jonathan Wight][schwa]
    unveiled a custom action for [Automator][Automator] which can run Python
    code.  His code uses [PyObjC][PyObjC] to allow the action to take
    advantage of all the Cocoa goodness that comes with it.  While I am
    doing more Python work these days, my heart still belongs to
    [Ruby][Ruby], so I was compelled to respond with a ["Run Ruby
    Script"][release] action of my own.
---

In a [recent blog post][RunPythonScript], [Jonathan Wight][schwa] unveiled a custom action for [Automator][Automator] which can run Python code.  His code uses [PyObjC][PyObjC] to allow the action to take advantage of all the Cocoa goodness that comes with it.  While I am doing more Python work these days, my heart still belongs to [Ruby][Ruby], so I was compelled to respond with a ["Run Ruby Script"][release] action of my own.

<img src="/images/RunRubyScript.png" alt="RunRubyScript.png" border="0" width="300px" align="left" style="margin-right: 8px"/>

The [source code][RunRubyScriptGIT] for this project is available under the [MIT license][MIT].  Like Jonathan's Python original, this action includes a (rather naïve) syntax highlighting NSTextView.  More on that perhaps in a future post.  But the real star of this action is the integration with Leopard's [Scripting Bridge][SB].

When Automator passes objects in to a custom Cocoa-based action, it converts them into one of a handful of possible Cocoa types.  Types such as strings (com.apple.cocoa.string), paths (com.apple.cocoa.path), and URLs (com.apple.cocoa.url) are happily handled for you.  Everything else can be sent as an Apple Event descriptor (com.apple.applescript.object).  That's all well and good, but I haven't the foggiest what to do when my script receives an NSAppleEventDescriptor.  What we really want is an object with methods and properties that can be used to access the underlying object directly.  With the help of Scripting Bridge, this is no problem at all.

The Scripting Bridge essentially wraps the scriptable interface of an application up into a nice Objective-C object.  With the help of [RubyCocoa][RubyCocoa], we can take advantage of those nicely wrapped interfaces in the "Run Ruby Script" action.  Since incoming data is passed in the form of an NSAppleEventDescriptor, the action performs some conversion in order to get a Scripting Bridge object for use by the script.  By recursively traversing the descriptor, we can glean the source application and attempt to create an SBApplication.  If that goes well, we work our way back up through the event descriptor, finding items in collections on our way back up.  This is best illustrated by an example:

Imagine that we ask iTunes to find all songs whose name equals "[The Salmon Dance][salmon]" (a [great song][salmon] recently recommended by [Cabel][Cabel]).  We might normally get an NSAppleEventDescriptor such as

<pre>
&lt;NSAppleEventDescriptor: 'obj '{ 
    'form':'ID  ', 'want':'cFlT', 'seld':3592, 'from':
        'obj '{ 'form':'ID  ', 'want':'cLiP', 'seld':3322, 'from':
            'obj '{ 'form':'ID  ', 'want':'cSrc', 'seld':42, 'from':
                'psn '("iTunes") } } }
&gt;
</pre>

I don't know about you, but my brain lost interest at `'want':'cFlT'`.  But let's not give up yet.  After invoking the powers of Scripting Bridge, we get

<pre>
&lt;ITunesFileTrack @0x12a5c40: 
    ITunesFileTrack id 3592 
    of ITunesLibraryPlaylist id 3322 
    of ITunesSource id 42 
    of application "iTunes" (98938)&gt;
</pre>

That gives me a warm fuzzy.

[Download][release] the action now!  If you find it useful or come up with an interesting use for it in a workflow, drop me a line or leave a comment here.  I'll be updating the repository periodically with improvements and examples.

**Words of warning**: There may be some issues if you have both the Ruby and Python script actions installed.  Specifically, Automator *may* crash if you try to add both actions into the same workflow.  I described the problem in brief in my article about [RubyCocoa bundles][RCBundle], and Bill Bumgarner described it in a bit more detail [on his blog][bbum].  The bug which causes this will hopefully be fixed in 10.5.2, but I may release a custom built version of RubyCocoa in the meantime if there is enough demand for it.

[release]: http://threeve.org/blog/releases/Run%20Ruby%20Script.action.zip
[RCBundle]: http://threeve.org/blog/2007/12/loadable-bundles-using-rubycocoa.html
[bbum]: http://www.friday.com/bbum/2007/11/25/can-ruby-python-an-objective-c-co-exist-in-a-single-application/
[SB]: http://developer.apple.com/documentation/Cocoa/Conceptual/ScriptingBridgeConcepts/Introduction/chapter_1_section_1.html
[MIT]: http://www.opensource.org/licenses/mit-license.php
[RunRubyScriptGIT]: http://code.threeve.org/?p=RunRubyScriptAction.git;a=summary
[Ruby]: http://www.ruby-lang.org/
[RubyCocoa]: http://rubycocoa.sourceforge.net/
[Automator]: http://www.apple.com/macosx/features/300.html#automator
[RunPythonScript]: http://toxicsoftware.com/run-python-script/
[schwa]: http://toxicsoftware.com/
[PyObjC]: http://pyobjc.sourceforge.net/
[Cabel]: http://cabel.name
[salmon]: http://phobos.apple.com/WebObjects/MZStore.woa/wa/viewAlbum?playlistId=257638042&s=143441&i=257638202
