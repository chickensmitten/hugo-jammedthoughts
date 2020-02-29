
+++
author = "Hugo Authors"
title = "Ruby on Rails Upgrades"
date = "2019-05-20"
description = "My foolproof algorithm for upgrading Ruby on Rails"
tags = [
    "ruby",
    "rails",
]
+++

# My foolproof algorithm for upgrading Ruby on Rails

[<img src="https://i.imgur.com/LEIghko.jpeg" width="100%;"/>](https://i.imgur.com/LEIghko.jpeg)

<img src="https://media.giphy.com/media/RkzMtKbCKFUY3wYRMy/giphy.gif" width="100%;" />

I was recently asked to upgrade a large RoR app from version 4.1 to 6.0. 
Having no previous experience upgrading Rails, quite immediately I found myself scouring the web for official and less-official guides, collecting notes and filtering what I thought were the essentials.

What I’ve come up with is my own take on the upgrade, it does *not*** **replace the official guide, but rather **builds on top of it** for smooth sailing.

The most important tip is to keep your spirit high; you will run into issues. And you will, slowly but surely, overcome them. Now let’s get started.

## Read the official upgrade guide

Reading through the [official upgrade guide](https://guides.rubyonrails.org/upgrading_ruby_on_rails.html) and the [target version release notes](https://guides.rubyonrails.org/5_0_release_notes.html) will help you tons. The latter is more of a recommendation, but the first is an absolute must.

**Why:** you will get acquainted with the process and the changes you need to make between versions, so when things start breaking it will hopefully not be as intimidating. The official guide also states which Ruby version you need for your target Rails version, and offers specific Rails version X to Y notes which will save you hours of chasing your tail.

## Have a green test suite with great coverage

Making sure you have a passing unit tests suite (such as Rspec) before beginning the upgrade is key to a safe upgrade — this can’t be stressed enough.

**Why: **a comprehensive test suite gives you peace of mind that your app is still working both after and during an upgrade. 
Whether you’re updating specific gems to match your new shiny Rails version, or changing production code to handle a deprecation — more on that ahead — **you will keep going back to the tests** to see they still pass.
This will boost your confidence tremendously, and confirm that you’re going in the right direction.

Good coverage tests will also raise deprecation warnings and will break in places where code needs to change. Without it you will have to manually test **every** flow in the app after you make a change.

## Upgrade gems incrementally

Following the conventions of [Semantic Versioning](https://semver.org/) (MAJOR.MINOR.PATCH), when upgrading gems, especially the rails gem, always move in increments to the **latest patch version of the current minor version**.



**Example: **Going from Rails version 4.1 to 5.0, the steps would be 4.1.11 (current version)>4.1.**16** (latest patch of 4.1)>**4.2.11.1** (latest patch of 4.2) >**5.0.7.2** and so on.

**Why: **It may seem confusing at first, but the rule of thumb is you don’t want to handle more than one significant change at a time.

Patch versions (4.1.**X**) only introduce backward-compatible bug fixes so you want those cleared out of the way first, and you can jump a few of them at a time without any real danger. 
Minor versions (4.**X**)** **are also backward-compatible, but can introduce new functionalities that you may want to adjust your code to, so it’s safer to do those one at a time.

You can use this [list of rails versions](https://rubygems.org/gems/rails/versions) as reference.

## Set explicit versions in your Gemfile

You may want to read a little about [bundle update](https://bundler.io/man/bundle-update.1.html) before proceeding.

Explicitly stating the dependency gems versions in your Gemfile will help you during the upgrade, since you will be able to slowly increment gems one-by-one to support a new Rails version.

**Why: **Although often recommended in forums, the convention of altogether removing the versions in your Gemfile, then running bundle update to upgrade Rails is dangerous when you have many dependency gems. More often than not, you will get too many gems changing together (and to much advanced versions than they must), and it’s easy to get lost.

**Example: **When you start your upgrade, your Gemfile may look something like this:

    source 'http://rubygems.org'

    gem 'rails', '4.2.11.1'
    group :test do
      gem 'rspec'
      gem 'factory_girl'
      gem 'database_cleaner'
    end
    
    gem 'mysql2', '~> 0.3.21'

You may notice that some gems don’t have versions next to them. Don’t worry— your Rails app knows exactly which version to use for each one, since the 
installed versions are locked in a Gemfile.lock file. More about it [here](https://stackoverflow.com/questions/6927442/what-is-the-difference-between-gemfile-and-gemfile-lock-in-ruby-on-rails/10959764).

Following the incremental approach, you decided to bundle your app with the next rails version — 5.0.7.2, so you make the following change:

    gem 'rails', **'5.0.7.2'**

After the change you run bundle update rails, but bundler fails on the **rspec** gem — your current version of rspec (3.4.0) only supports rails < 4.1.

You could now run bundle update, which will update all gems without version to the **latest** compatible version with this Rails version, which will probably be enough for your app to bundle.

However,** **as warned above, your gem versions may jump too far ahead and introduce unnecessary issues (for example, a gem which has implicitly jumped a major version may break your code).

Instead, we’re going to “nail” gem versions to the ones already in the Gemfile.lock, and then updating with the smallest increments, gem by gem. This will save you a lot of headache.

**Example: **using your Gemfile.lock, add versions to all gems in your Gemfile. Don’t change the rails gem version yet. In the end it should look like this:

    source '[http://rubygems.org'](http://rubygems.org')

    gem 'rails', '4.2.11.1'

    group :test do
      gem 'rspec', '3.4.0'
      gem 'factory_girl', '4.8.0'
      gem 'database_cleaner', '1.6.1'
    end

    gem 'mysql2', '0.3.21'

Don’t worry, you only have to do this step once.
To verify you’ve done it correctly, run bundle update and see that no gems have updated (their version were just made explicit, not changed).

Now it’s time to change the rails version and run bundle update rails.

**rspec** still doesn’t bundle, but now it’s easy to see its version is 3.4.0. 
What I like to do at this point is to [check the target rails release date](https://rubygems.org/gems/rails/versions), then find a gem version for the failing gem which was released around the same time.

In our example: Rails** 5.0** was released in **June 30, 2016**. 
Checking the [rspec gem version in RubyGems](https://rubygems.org/gems/rspec/versions) shows a new minor version came out in July 1, 2016– version **3.5.0**. 
So we’ll update this gem version in Gemfile:

    gem 'rspec', '3.5.0'

Now run bundle update rails rspec. The app should successfully bundle, or prompt about the next out-of-date gem.

Note that some gems have upgrade notes for upgrading to a new rails version, so definitely check the gem’s Github page before doing the guesswork.

## Commit early and often

After successfully bundling a previously failing gem, or making a code change in the repo, make sure tests are passing and commit to your git branch.

**Why: **Sometimes, while you’re already a few hours in, you find that some previously working flow started failing. Having a working baseline you can return to can relieve a lot of stress, since you can easily find the exact spot where you introduced the change that broke things.

Rolling back easily is key to regaining confidence and starting over fresh when sh*t hits the fan.

<iframe src="https://medium.com/media/96c038e26944e2c71295f09412745d74" frameborder=0></iframe>

## Deprecation warnings are your friend

When upgrading Rails, you will inevitably see warnings in the console such as:

    DEPRECATION WARNING: confirm! is deprecated in favor of confirm (called from ... at /app/models/user.rb:123).

They may come from Rails itself, or one of its dependency gems (the above example comes from the **devise** gem). Don’t ignore these messages! They are made for you, dear upgrader.

**Why: **deprecation warnings give you a heads-up before something is about to change in the next versions, so following them and adjusting the code respectively as soon as they arise will ensure the app won’t break on the next version bump.

### Bonus: tracking an outdated gem

If you see a deprecation warning about a line of code which you can’t find in your repo, it is probably because you have an outdated gem which is calling it.
A nice trick is to run the following command:

    ack <code causing the warning> `bundle show rails`/..

This basically does a code search in your installed gems source code. Once you find the culprit gem, upgrading it will usually solve the deprecation warning.

## Use railsdiff

According to the official upgrade guide, a part of the upgrade is to run an update task (rails app:update, or rake rails:update before 4.2).
The interactive prompt will walk you through changed configurations and allow you to pick the ones you want.
**I found this less effective on a large app, **where you have multi configuration environments.

Alternatively, [railsdiff](http://railsdiff.org) is a great tool for seeing code differences between Rails versions. The tool will show you which files have changed in a nice view that resembles Github, so you can decide which changes you want to cherry pick in your upgrade.

**Example: [**http://railsdiff.org/4.2.11.1/5.0.7.2](http://railsdiff.org/4.2.11.1/5.0.7.2)
Use it as a supporting tool to the techniques mentioned above.

## Conclusion

Upgrading a Rails version can be a daunting task. Following the above tips should set you up for working incrementally, achieving “small wins” until you suddenly find yourself with a fully bundled app, your test suite passing with flying colors, and before you know it - you’re done :)
 
I hope this article helps you with your upgrade, and that it goes smoothly as possible. Got any tips from your upgrade experience?
Come back and leave them in the comments section below. Good luck!

<iframe src="https://medium.com/media/ce2862530a7b7aba400da942e994cb5b" frameborder=0></iframe>
