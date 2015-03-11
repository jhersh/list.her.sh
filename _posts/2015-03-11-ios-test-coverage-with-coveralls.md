---
layout: post
title: iOS Test Coverage with Coveralls
summary: Measuring code coverage in your iOS app or library for fun and profit!
tags: ios
---

Code coverage measures how much of your app or library is executed by at least one of your tests. This manifests as a percentage from 0-100% (higher is better!) that can be a good high-level indicator of the quality — or at least the breadth — of your tests. 

Today we'll see how easy it is to automatically measure, report, and visualize code coverage in your iOS app or library. You'll end up with a report [like this one](https://coveralls.io/builds/1975865) for each file in your project and a great visual representation showing line-by-line where your coverage is strong, where you can improve, and how your coverage trends up (or down!) over time. 

You will need:

1. An iOS app or library to test
2. Some tests to execute against your app or library
3. A CI service that executes your tests, like [Travis CI](https://travis-ci.org) or [CircleCI](https://circleci.com), both of which are free for open-source projects
4. A code coverage tool like [Coveralls](https://coveralls.io) that can visualize your code coverage results. Coveralls is free for open-source projects!
5. A utility like [slather](https://github.com/venmo/slather) that sends your test code coverage results from your CI service (#3) to your code coverage tool (#4) every time your CI service runs your tests

[SSDataSources](https://github.com/splinesoft/SSDataSources), [Jazz Hands](https://github.com/IFTTT/JazzHands), and [FastttCamera](https://github.com/IFTTT/FastttCamera) are great examples of open-source iOS projects that report code coverage with Travis CI, Coveralls, and slather. As of this writing, slather does not yet support CircleCI, but work is [in progress](https://github.com/venmo/slather/pull/55) and SSDataSources is [nearly there](https://github.com/splinesoft/SSDataSources/pull/49).

First, you'll need to get CI to run your project's tests each time you push a commit. See the [Travis CI iOS docs](http://docs.travis-ci.com/user/languages/objective-c/) and [CircleCI iOS docs](https://circleci.com/docs/ios), or perhaps you'll be inspired by these [`.travis.yml`](https://github.com/IFTTT/JazzHands/blob/master/.travis.yml) and [`circle.yml`](https://github.com/splinesoft/SPLUserActivity/blob/master/circle.yml) files from the above projects. 

Now you'll need to add the [slather](https://github.com/venmo/slather) rubygem to your project. You don't necessarily need slather installed locally on your development machine; rather, you could include it in your project's `Gemfile` or as a dependency command [like this](https://github.com/IFTTT/JazzHands/blob/master/.travis.yml#L3) in your `.travis.yml`. You'll also need to instruct your CI service to execute slather after a successful build; for Travis CI, you might add to your `.travis.yml` a line like `after_success: slather`.

Next you'll need to update some build settings in your project so that when your project's tests are run, `xcodebuild` will measure the code paths that are executed by your tests and generate a file that code coverage utilities can parse for coverage results.

There are three ways to set this up. Pick your favorite:

1. If you have slather installed on your development machine, you can execute `slather setup path/to/project.xcodeproj` to enable the necessary settings in your Xcode project. 
2. If you don't have slather locally, head to the "Build Settings" tab of your Xcode project, scroll down to the "Code Generation" section, and set to YES these two settings for the scheme(s) and configuration(s) that your tests will execute: **Generate Test Coverage Files** and **Instrument Program Flow**.
3. You could update the flags passed to `xcodebuild` in the [build command](https://github.com/IFTTT/JazzHands/blob/master/.travis.yml#L10) in your `.travis.yml` or `circle.yml` to add: 

{% highlight bash %}
GCC_INSTRUMENT_PROGRAM_FLOW_ARCS=YES 
GCC_GENERATE_TEST_COVERAGE_FILES=YES
{% endhighlight %}

You might consider updating your CI script's build flags (#3 above) even in addition to #1 or #2 to be extra certain that your CI service is generating code coverage files for every build.

Next, head to [Coveralls](https://coveralls.io), sign in with your Github user, and [add your repo](https://coveralls.io/repos/new) to Coveralls.

Your project will also need a simple `.slather.yml` file in the root of your repo. This instructs slather how to process your code coverage results and where to send them. Here is one of mine, for use in a Travis CI project:

{% highlight yaml %}
ci_service: travis_ci
coverage_service: coveralls
xcodeproj: Example/ExampleSSDataSources.xcodeproj
source_directory: SSDataSources
{% endhighlight %}

Of particular note, particularly for CocoaPods repos, is the `source_directory` parameter. Here you can specify exactly which files should be included in your code coverage results — this is necessary so that you can measure only the files in your pod and exclude source files that might be in a sample or demo project.

That's it! Package up a commit with your changes, send it via carrier pigeon to Github, and — if your build succeeds — you should see your very first coverage report on Coveralls.
