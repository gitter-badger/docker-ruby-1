# [jdickey's Customised Ruby Builds](https://github.com/jdickey/docker-ruby)

# Contents


# Overview

I *often* build from Ruby [official base images](https://hub.docker.com/_/ruby/), install additional software packages, and do some basic Ruby housekeeping (installing Bundler and making sure the system Gems are up-to-date). On a reasonably modern iMac with a decent First World internet connection, this can take about *20 minutes.* Repeat this half-a-dozen times over the course of a day and you've lost two hours. As Orwell wrote, *doubleplus ungood.*

## IMPORTANT NOTES

### For Versions 0.15.0 and Later

Starting from Version 0.15.0, the `capybara-webkit` Gem (and its supporting `json` Gem) **will no longer be part of this image**. It's given us and countless other teams ulcers for quite some time, and our own downstream projects no longer use it (in favour of Capybara+Selenium presently, but [Rubium](https://github.com/vifreefly/rubium)) is firmly in the "assess" ring of our technology radar.)

Does this mean that we'll stop building Qt-including images entirely? Very possibly at some point after 0.15.0, as the only major remaining Gem installed is `capybara`, which doesn't itself even _require_ Qt. Our feeling is that simplifying this image's build management by pushing installation of `capybara` and Qt to higher-level images that actually _use_ them would be a net positive for us.

If there are any other users of these images with strong opposing opinions, now is the time to get that discussion started.

### For Images Prior To Version 0.14.0

Basically, *please do not use them.* Rebuild any of your images using `jdickey/ruby` as a base using the current-at-the-time-of-writing Version 0.14.0 or later.

Version 0.14.0 includes images for Ruby versions **2.5.3** and **2.4.5**, as well as a subset of previous releases in the 2.5.*x* and 2.4.*x* for completeness. **You are urged in the strongest possible terms _not_ to use any images whose tags contain `2.5.0` or `2.5.1`, or `2.4.0` through `2.4.4` inclusive, for any purpose other than testing.** (The upstream base image for `2.5.2` has been yanked.) Do **not** deploy your own images based on known-insecure Ruby versions where others can run them, particularly where assumptions are being made about the security and encryption offered by SSL. See the [release notes](https://www.ruby-lang.org/en/downloads/releases/) for specific Ruby versions to learn more.

A comparable notice for the release of Version 0.13.0 read

> The Alpine images are believed to be OK, but the Debian images, even those claiming to support Qt5, in fact have a mixture of Qt4 and Qt5 which causes several versions of `capybara-webkit` to have Issues, and which imminent future versions of `capybara-webkit` will not support, as they've officially deprecated Qt4. To find which version of the image you're working with, run the command line `docker inspect jdickey/ruby:2.5.1 | grep '"version"'`, substituting the tag of the image you are actually using if not `2.5.1`.

## Supported Tags

Each image has one tag that follows the format `2.x.y-os_build[-no-qt]`, where

1. `2.x.y` is the full version number of the Ruby version hosted by the image, which will be one of `2.5.3` (the current version), `2.5.1`, `2.5.0`, `2.4.5` (the current `2.4` release), or `2.4.4`. Again, you are **strongly urged** not to use releases prior to the latest release on a given version branch (`2.5` or `2.4`) for new images;
2. `os_build` identifies which OS and variant the image was based on. These can be any one of
	1. `stretch`: Debian [Stretch](https://en.wikipedia.org/wiki/Debian#Code_names) (9.0);
	2. `stretch-slim`: A "slim" version of Stretch;
	3. `alpine3.7` (synonyms: `alpine37` and `alpine`): Alpine Linux 3.7, a minimalist Linux distribution. (There is no 'slim' version of `alpine`; it's already the smallest of the listed images);
3. The suffix `-no-qt` indicates that the image has been built *without* the Qt OS-level libraries and tools needed to run the [Capybara](https://teamcapybara.github.io/capybara/) test framework (and thus does not include Capybara itself or the `capybara-webkit` headless browser).

The `latest` tag identifies the latest version of Ruby (as of October 2018, version 2.5.3) on the latest, non-`slim` version of Debian (currently `stretch`), built *with* the Qt and Capybara tools. It should be the default choice when you simply want the most recent supported Ruby version, but is not recommended for production use in most cases. (We recommend exploring basing your image on an `alpine` and `-no-qt` build for production.)

Minor-version tags such as `2.4-stretch` or `2.5-alpine-no-qt` identify the latest supported release of that minor version of Ruby on the specified OS build. As later versions of Ruby are released and supported (e.g., a hypothetical 2.4.6 or 2.5.4), the corresponding minor-version tag will be redefined to match the new full-version-number tag.

Major-version tags (e.g., `2-stretch` or `2-alpine3.7`) identify the latest supported minor release of the latest supported major release of Ruby (currently `2.5.1`), and are updated in a manner analogous to minor-version tags; i.e., when Ruby 2.6 is released (expected to be in December, 2018), each major-version tag will be updated to match `2.6.0` on each respective operating system (e.g., `2.6.0-stretch`).

Tags which *do not* include a Ruby version number are built using the latest supported version of Ruby on the OS build identified by their name, e.g., `stretch-no-qt` or `alpine`, built with Qt and Capybara. This differs from `latest` in that `latest` is presently defined as always being on the latest non-`slim` Debian build (currently `stretch`).

Finally, the Alpine Linux OS build names differ from the Debian conventions in that Alpine is the only OS build that encodes (major and minor) version numbers. Hence, `alpine3.7` is synonymous with `alpine37` and, until a newer version of Alpine is supported by the [official Ruby base images](https://hub.docker.com/_/ruby/), `alpine` is as well. That is to say that `2.5.1-alpine3.7` and `alpine` (presently) reference the same image.

### Logical but Nonexistent Tags

Why no tags for, e.g., `2-slim-jessie`? First, support for Jessie is dead; as of September, 2018, all Debian builds are on Stretch. We use tag names intended to be consistent with those of the [official Ruby Docker images](https://hub.docker.com/_/ruby/); the unadorned `2` states that the tag is for the latest 2.*x* Ruby version. No Debian Jessie-based images should exist for any Ruby version. It is possible that some such tags were created in error and not caught prior to release on Docker Hub; if you find one, please [open an issue](https://github.com/jdickey/docker-ruby/issues). *Thanks!*

### What? No Dockerfiles?

Versions of these images prior to [Release 0.11.0](#changelog) published `Dockerfile`s for each supported Ruby version/OS build variant. Release 0.11.0 and later use [Docker build arguments](https://docs.docker.com/engine/reference/commandline/build/#set-build-time-variables---build-arg) to configure the appropriate "template" `Dockerfiles` instead, depending on whether a Debian or Alpine image is being built, with or without Qt and Capybara. These templates are:

* [`Dockerfile.main.alpine`](https://github.com/jdickey/docker-ruby/blob/master/Dockerfile.main.alpine);
* [`Dockerfile.main.debian`](https://github.com/jdickey/docker-ruby/blob/master/Dockerfile.main.debian);
* [`Dockerfile.no-qt.alpine`](https://github.com/jdickey/docker-ruby/blob/master/Dockerfile.no-qt.alpine); and
* [`Dockerfile.no-qt.debian`](https://github.com/jdickey/docker-ruby/blob/master/Dockerfile.no-qt.debian).

# Software

## Debian Stretch

Jessie is dead; long live Stretch! Ancient, now-unsupported upstream official Ruby images were based on Debian Jessie; as of Version 0.13.2 of these images, we no longer build Ruby images based on no-longer-supported Ruby versions (2.4.3 and earlier).

### Base Software

The following Debian packages are installed in Debian-based images of this repo:

* `build-essential`
* `curl`
* `nodejs`
* `sudo`
* `wget`
* `zsh`

### Qt5 Software

The following Debian packages are installed in all Debian images not tagged `no-qt` of this repo. This is useful for test-mode builds that include tools such as [Capybara](https://github.com/teamcapybara/capybara):

* `libqt5webkit5-dev`
* `qt5-qmake`
* `libqtwebkit-dev`
* `libxml2-dev`
* `libxslt1-dev`
* `xvfb`

## Alpine Linux

### Base Software

The following Alpine packages are installed in Alpine-based images of this repo:

* `alpine-sdk`
* `bash`
* `build-base`
* `nodejs`
* `tzdata`
* `zsh`

### Qt5 Software

The following Alpine packages are installed in Alpine images not tagged `no-qt` of this repo. This is useful for test-mode builds that include tools such as [Capybara](https://github.com/teamcapybara/capybara):

* `libxml2-dev`
* `libxslt-dev`
* `qt5-qtwebkit-dev`
* `xvfb`

# Changelog

## 0.14.2 (28 December 2018)

To (probably) close out the old year, we've updated Gems again (including `bundler` and RubyGems itself). There isn't yet an official Ruby 2.6.0 upstream image yet; that will be the base for 0.15.0 as soon as it's available.

## 0.14.1 (20 December 2018)

* Added "Important Notes" subsection _For Versions 0.15.0 and Later_; **please read**;
* Updated Gem versions.

## 0.14.0 (19 October 2018)

* Introduced images built on Ruby versions 2.5.3 and 2.4.5.
* Reworded the README to emphasise the urgency of using outdated Ruby versions only for testing.

## 0.13.4 (10 October 2018)

* No images based on Debian Jessie are supported as of this release;
* Installed (then-)current version of Bundler into default Gem set;
* Updated now-outdated `capybara`, `capybara-webkit`, and `nokogiri` Gems.

## 0.13.3 (24 September 2018)

* Added new `jdickey_ruby_image_version` label with same content as `version`. The latter may (should?) be overwritten by subsequent images using this as a base; the former ought not to be;
* Updated now-outdated Gem versions.

## 0.13.2 (20 September 2018)

* Ruby versions prior to 2.4.4 are *no longer built as images*. Consequently, all upstream images based on Debian support Stretch, and so all Jessie builds (as well as Alpine builds of Ruby versions prior to 2.4.4) have been dropped.

## 0.13.1 (24 June 2018)

* Ruby <= 2.4.4 on Debian Jessie fixed upstream (but still to be removed on or after 11 September 2018).

## 0.13.0 (11 June 2018)

* Corrected Qt5 library specification, eliminating Qt4.
* Simplified image-build process.
* Added deprecation notice for Ruby < 2.4.4 to this README.

## 0.12.0 (11 April 2018) WITHDRAWN &mdash; DO NOT USE

## 0.11.2 (15 March 2018)

* Corrected ordering of name tokens in Debian 'slim' images. (Fixes Issue #4.)
* Fixed some minor errors in tagging.
* Added, then commented out, building older Ruby versions on Debian Stretch. (See Issue #5.)

## 0.11.1 (7 March 2018)

* Added `2.5.0-alpine37` as synonymous tag for `2.5.0-alpine-3.7`
* Added `2.5.0-alpine37-no-qt-` as synonymous tag for `2.5.0-alpine-3.7-no-qt`

## 0.11.0 (4 March 2018)

* **BREAKING CHANGE:** Slim images now named consistently with corresponding official Ruby images; e.g., rather than `2.5.0-stretch-slim`, that image is now tagged `2.5.0-slim-stretch`
* Rearchitected build process to no longer require a separate `Dockerfile` per image being built
* Made base image specs and as-generated image tag more explicit
* Added `yard` and `yard config --gem-install-yri` to `-no-qt` images
* Moved `libxml2-dev` and `libxslt1-dev` packages from `-no-qt` images to Qt/Capybara images, as they are required by Capybara rather than other `-no-qt` packages

## 0.10.0 (1 March 2018)

* Added Ruby 2.4.3 (on Debian Jessie) images
* Reworded `description` label
* Cleaned up package-manager command lines

## 0.9.0 (11 January 2018)

* Qt-included images based on `-no-qt` images, rather than directly on official base image
* Added `LABEL` metadata to built images
* Further cleanup of `Dockerfile`s and build scripts

## 0.8.0 (8 January 2018)

* Added Ruby 2.5.0 (on Debian Stretch and Alpine 3.7) images
* Improved consistency between tag names

## 0.7.0 (17 November 2017)

* Initial release of 2.4 (actually 2.4.2) on Debian Jessie images.

# Additional Documentation

See the [official Ruby Docker image docs](https://hub.docker.com/_/ruby/).

# Legal

All files in this repository are Copyright &copy; 2017 by Jeff Dickey, and licensed under the [MIT License](https://opensource.org/licenses/MIT).

As with all Docker images, these likely also contain other software which may be under other licenses (such as Bash, etc from the base distribution, along with any direct or indirect dependencies of the primary software being contained).

As for any pre-built image usage, it is the image user's responsibility to ensure that any use of this image complies with any relevant licenses for all software contained within.
