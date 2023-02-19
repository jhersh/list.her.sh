---
layout: post
title: The right xcodebuild at the right time
summary: xcodebuild, I choose you!
tags: ios
---

What an incredible WWDC! You've probably been checking out the new bits and you've collected several versions of Xcode on your machine. 

Traditionally, you'd switch active `xcodebuild` versions using the `xcode-select --switch` command-line utility. But this requires `sudo` and switches the active developer directory system-wide, both of which can be undesirable particularly for CI builds.

It turns out you can specify a per-build developer directory on the command line without `xcode-select`! The trick is twofold: assign a value to the `DEVELOPER_DIR` environment variable and run your build with `xcrun`.

I standardize my build commands with `rake` tasks. Here's a `rake test` task you might use, perhaps in your `xcode-7` branch, to execute a single build using an alternate Xcode:

{% highlight ruby %}

desc 'Run all application tests.'
task :test do

  dev_dir = ENV['DEVELOPER_DIR'] || ""
  
  # Path to desired Xcode beta
  ENV['DEVELOPER_DIR'] = '/Applications/Xcode-beta.app/Contents/Developer'
  
  sh "set -o pipefail && xcrun -sdk iphonesimulator "+
  "xcodebuild ... etc ... "+
  "clean test | bundle exec xcpretty -ct"

  ENV['DEVELOPER_DIR'] = dev_dir
  
end

{% endhighlight %}
