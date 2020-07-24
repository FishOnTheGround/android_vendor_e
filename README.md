# Classic / Standard Android build process

## Why *not* simply using "Docker"?

The provided Docker image is the recommended way to build /e/.
Especially for new users who never build Android before it is THE method
which will simply just work. It does not require a Linux setup and has
extreme low barriers for beginners.

If you ask yourself if you should go the recommended way or the classic
one the anywer would be almost always: use Docker instead of this here.

Checkout the [docker guide][docker-guide] for that.

## Who might wanna use this repo?

If you can answer any or all of the following with yes, then this repo
is for you:

 - you have build Android before and are used to the classic way of building
 - you have already a good working build environment
 - you do not like docker :P
 - you want to get the most speed possible
 - you want to learn how Android would be normally build
 - you plan to build other Android ROMs like LOS, AOSiP etc and do not want
   to start everytime from scratch
 - you already have Linux, have good experience with Linux and/or be willing
   to get used to it :)

Some / all of the above matches on you? Well then this is for you!


## Getting started

To get started with Android and/or /e/, you'll need to get
familiar with [Repo](https://source.android.com/source/using-repo.html) and [Version Control with Git](https://source.android.com/source/version-control.html).

To initialize your local repository using the /e/ trees, use a command like this:
```
repo init -u https://gitlab.e.foundation/e/os/android.git -b <branch>
e.g:
repo init -u https://gitlab.e.foundation/e/os/android.git -b v1-pie
```
Then add this repo to your local manifest:
```
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
    <project path="vendor/e" name="steadfasterX/android_vendor_e" remote="e" revision="v1-pie" />
<manifest>
```
Finally sync the sources:
```
repo -j<processes> sync
e.g.
repo -j8 sync
```


## Setup /e/

### vendorsetup.sh (your device tree)

you find that one here: `device/<vendor>/<codename>/vendorsetup.sh`

if it does not exists (might be the case on newer tree's where the lunch combo's have been moved already) just create it.

you need at least to set 1 variable here:

 - `EOS_DEVICE`: your device's codename. that means set it identical to what you have defined for "PRODUCT_DEVICE".
 
 there are more variables you *can* set here (optional), some maybe interesting examples are:

 - `EOS_SIGNATURE_SPOOFING`: add or add not microG, or add it restricted (see topic "Signature spoofing")
 - `EOS_BRANCH_NAME` the [release branch][release-branches] you want to build on, e.g. "v1-pie"
 - `EOS_RELEASE_TYPE`: the type of your release, e.g. "UNOFFICIAL"
 - `EOS_CUSTOM_PACKAGES`: override the list of /e/ apps to be included

for a complete list and their default setting look in vendor/e/[vendorsetup.sh][vendorsetup].

note: do not change variables there! just find out what the proper variable name is
and set in your `device/<vendor>/<codename>/vendorsetup.sh`!


### Signature spoofing

If you want to make use of [microG][microg] there are 2 options for the required [signature spoofing patch][signature-spoofing]:

 * "Original" [patches][signature-spoofing-patches]
 * Restricted patches

With the "original" patch the FAKE_SIGNATURE permission can be granted to any
user app: while it may seem handy, this is considered dangerous by a great
number of people, as the user could accidentally give this permission to rogue
apps.

A more strict option is the restricted patch, where the FAKE_SIGNATURE
permission can be obtained only by privileged system apps, embedded in the ROM
during the build process.

The signature spoofing patch can be optionally included with:

 * `EOS_SIGNATURE_SPOOFING`: `yes` to use the original patch, `restricted` for
    the restricted one, `no` for none of them

If in doubt, use `restricted`: note that packages that requires the
FAKE_SIGNATURE permission must be embedded in the build by adding them in

 * `EOS_CUSTOM_PACKAGES`

Extra packages can be included in the tree by adding the corresponding manifest
XML to the local_manifests volume.

### Proprietary files

Some proprietary files are needed to create a LineageOS build, but they're not
included in the repo for legal reasons. You can obtain these blobs in
four ways:

 * by [pulling them from a running LineageOS][blobs-pull]
 * by [extracting them from a LineageOS ZIP][blobs-extract]
 * by downloading them from [TheMuppets repos][blobs-themuppets] (unofficial)
 * by adding them to a local_manifests definition (e.g. roomservice.xml)

The fourth way is enabled by default; if you're OK with that just move on,
otherwise set `EOS_INCLUDE_PROPRIETARY` to `true` in `device/<vendor>/<codename>/vendorsetup.sh` to pull them from TheMuppets automatically.

### OTA

If you plan to provide updates for your unofficial and/or custom build you can
setup your own [custom OTA server][customOTA]. 

Follow that guide to setup and prepare your device tree to use it.


### Signing

By default, builds are signed with your own keys - created automatically on build.
If you want to sign your builds with the default test-keys (**not recommended**) just
setup your device tree vendorsetup.sh with:

 * `EOS_SIGN_BUILDS`: set to `false` to sign the builds with the test-keys


## Start building

This repo provides 2 new build targets:

`mka eos`:

    does a full build without any sync

`mka eos-fresh`:

    does a full build but sync all sources before
    (think of it like a git reset --hard on any repo + sync everything
    so never use that if you have any uncommitted local changes..)


`mka eos` is the closest to a classic brunch/mka otapackage and should
be used by default.

so a complete run would be:

~~~
source build/envsetup.sh
lunch <your-device>
mka eos
~~~

## Finally

enjoy your new created build and do not forget to share your work at the [e foundation forum][e-forum]


[e-forum]: https://community.e.foundation/
[docker-guide]: https://community.e.foundation/t/howto-build-e/
[release-branches]: https://gitlab.e.foundation/e/os/releases/-/branches
[customOTA]: https://community.e.foundation/t/howto-create-your-custom-ota-server/19154
[vendorsetup]: vendorsetup.sh
[signature-spoofing]: https://github.com/microg/android_packages_apps_GmsCore/wiki/Signature-Spoofing
[microg]: https://microg.org/
[signature-spoofing-patches]: src/signature_spoofing_patches/
[blobs-pull]: https://wiki.lineageos.org/devices/bacon/build#extract-proprietary-blobs
[blobs-extract]: https://wiki.lineageos.org/extracting_blobs_from_zips.html
[blobs-themuppets]: https://github.com/TheMuppets/manifests
