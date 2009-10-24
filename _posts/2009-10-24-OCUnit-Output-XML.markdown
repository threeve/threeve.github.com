---
title: OCUnit XML File Output
tags: ['Cocoa', 'Testing', 'OCUnit']
layout: post
summary: >
    In the world of Mac OS X programming, OCUnit reigns supreme as the unit
    testing library of choice.  Apple has put a great deal of work into
    ensuring that OCUnit is tightly integrated into the standard development
    workflow with Xcode.  Unfortunately many other tools--such as continuous
    integration servers or test metric tools--expect test output in an XML
    file, typically in the format of the jUnit output from the Java Ant build
    tool.  OCunit does not provide xml file output by default, but adding
    support for it is quite easy.
---

In the world of Mac OS X programming, OCUnit reigns supreme as the unit
testing library of choice.  Apple has put a great deal of work into
ensuring that OCUnit is tightly integrated into the standard development
workflow with Xcode.  Unfortunately many other tools--such as continuous
integration servers or test metric tools--expect test output in an XML
file, typically in the format of the jUnit output from the Java Ant build
tool.  OCunit does not provide xml file output by default, but adding
support for it is quite easy.


The Goal
--------

Build servers such as [Hudson][] have the ability to process test
output and generate trend graphs and notifications.  Most such tools rely
on having the output in a particular format, often the jUnit format.  OCUnit
by default only prints its output to the console. 

The jUnit XML format produced by Ant is horribly undocumented.  It does
seem to be a rather simple format though, which basically looks like
this:

    <testsuites>
        <testsuite name="MyTestSuite">
            <testcase name="MyTestCase.testFooMethod"></testcase>
            <testcase name="MyTestCase.testBarMethod">
                <failure>Exception, blah blah blah</failure>
            </testcase>
        </testsuite>
    </testsuites>


The Means
---------

OCUnit does offer a few hooks which can be used to follow test
execution.  A quick peek around the SenTestingKit framework
headers--found in
`/Developer/Library/Frameworks/SenTestingKit.framework/Headers`-- shows
a couple of obvious ways to watch test execution.  The first is the
`SenTestObserver` class, and the second is a collection of 
`NSNotifications` that `SenTestObserver` is presumably based upon.

#### SenTestObserver

The `SenTestObserver` class defines a set of methods that will be
called at the beginning and end of both test suites and test methods.
This looks like a perfect solution, but it is completely non-obvious how
to actually use a subclass of `SenTestObserver`.  Also, there are at
least some reports that [`SenTestObserver` is broken][so1].


#### Notifications

Since there appear to be some issues with using `SenTestObserver`
directly, the next step is to directly observe the set of notifications
upon which it is based.  The following notifications are defined in the
SenTestingKit headers:

* `SenTestSuiteDidStartNotification`
* `SenTestSuiteDidStopNotification`
* `SenTestCaseDidStartNotification`
* `SenTestCaseDidStopNotification`
* `SenTestCaseDidFailNotification`

Using these notifications it is possible to create a class that can
write to an XML file as test execution progresses.  The only interesting
task remaining is to load the class before test execution starts, which
is easily accomplished using a [gcc constructor attribute][constructor].

The Result
----------

`BPOCUnitXMLReporter` is a small bit of code that records OCUnit test
execution and writes it to an XML file.  The code, along with a small
sample project, is [available on github][BPOCUnitXMLReporter] and can 
be easily added to any OCUnit test bundle.  Simply add the
`BPOCUnitXMLReporter.m` file to the test bundle target, and every time
the tests run a file `ocunit.xml` will be written with the test results.
Contributions are certainly welcome, so fork away!


[Hudson]: http://hudson-ci.org/
[so1]: http://stackoverflow.com/questions/247607/how-do-i-trap-ocunit-test-pass-failure-messages-events/329674#329674
[BPOCUnitXMLReporter]: http://github.com/threeve/BPOCUnitXMLReporter
[constructor]: http://developer.apple.com/mac/library/documentation/DeveloperTools/gcc-4.0.1/gcc/Function-Attributes.html
