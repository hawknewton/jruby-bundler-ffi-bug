Overview
========
It would appear jruby 1.7.13 (and up) preloads the ffi gem as either part of
it's initialization process or the initialization process of bundler.  This
behavior differs from other versions <= 1.7.12.

I've tested up to 1.7.15, at the moment.

Who cares?
----------
This normally isn't a problem unless you're trying to use bundler's
`--deployment` flag and your runtime doesn't exhibit this behavior.

Behold:

```bash
➜  jruby-bundler-ffi-bug  ruby --version
jruby 1.7.13 (2.0.0p195) 2014-06-24 43f133c on Java HotSpot(TM) 64-Bit Server VM 1.7.0_51-b13 [darwin-x86_64]

➜  jruby-bundler-ffi-bug  bundle --deployment
Using rake 10.3.2
Using ffi 1.9.3
Using bundler 1.7.2
Your bundle is complete!
It was installed into ./vendor/bundle

# When using jruby-1.7.13, everything works as expected
➜  jruby-bundler-ffi-bug  bundle exec rake
ffi loaded successfully!

# Switch to 1.7.12
➜  jruby-bundler-ffi-bug  rbenv shell jruby-1.7.12
➜  jruby-bundler-ffi-bug  ruby --version
jruby 1.7.12 (2.0.0p195) 2014-04-15 643e292 on Java HotSpot(TM) 64-Bit Server VM 1.7.0_51-b13 [darwin-x86_64]

# Things break
➜  jruby-bundler-ffi-bug  bundle exec rake
Could not find ffi-1.9.3-java in any of the sources
Run `bundle install` to install missing gems.

# ffi is not in vendor/bundle
➜  jruby-bundler-ffi-bug  ls -l vendor/bundle/jruby/1.9/gems
total 0
drwxr-xr-x  17 hnewton  staff  578 Sep  4 05:57 rake-10.3.2

# When running bundle --deployment with 1.7.12 it is
➜  jruby-bundler-ffi-bug  bundle install --deployment
Fetching gem metadata from https://rubygems.org/...........
Using rake 10.3.2
Installing ffi 1.9.3
Using bundler 1.7.2
Your bundle is complete!
It was installed into ./vendor/bundle

➜  jruby-bundler-ffi-bug  bundle exec rake
ffi loaded successfully!

➜  jruby-bundler-ffi-bug  ls -l vendor/bundle/jruby/1.9/gems
total 0
drwxr-xr-x   6 hnewton  staff  204 Sep  4 06:03 ffi-1.9.3-java
drwxr-xr-x  17 hnewton  staff  578 Sep  4 05:57 rake-10.3.2
```

Workarond
---------
Either don't execute `bundle --deployment` under 1.7.13 or make sure your
runtime uses 1.7.13.  This is an issue for me because I'm using torquebox
and don't directly control the version of the jruby runtime.

One might argue this isn't actually a bug and that you should always deploy
using the same version of jruby you bundle'd under, and such an argument
has merit.  I might counter that as a patch release behavior should remain
backwards compatible and it certainly violates the
[Principal of least surprise](http://en.wikipedia.org/wiki/Principle_of_least_astonishment)
as it surprised the hell out of me.

Next Steps
----------
* Test with jruby > 1.7.15
* Create a patch and issue a PR against jruby/jruby
