# rattlesnakeos-patches
My patches for Rattlesnake OS.

## opengapps
Note that building opengapps with Rattlesnake OS will require a larger EC2 volume size. By default it's set to 200 GB, but can be increased by editing the value of the `VolumeSize` key in `templates/lambda_template.go`. 400 worked for me.

As well, opengapps' aosp_build requires `zlib` I created a script to install it, available at https://github.com/corrmaan/rattlesnakeos-scripts. `git-lfs` is also required but I didn't have luck with having it installed through the script, so I made a `rattlesnakeos-stack` patch for it, available [here](https://gist.githubusercontent.com/corrmaan/5966d62f050d0f0cf22d7af60c37aac3/raw/50da4db5386bf791feda293db913741530e956f1/rattlesnakeos-stack-10.0.6-opengapps.patch "rattlesnakeos-stack-10.0.6-opengapps.patch"). After cloning `rattlesnakeos-stack`, apply the patch and build from source:

```bash
git clone https://github.com/dan-v/rattlesnakeos-stack.git
cd rattlesnakeos-stack
git checkout 794d5887d8e05ddcffb2d061204bd726c123e52e # 10.0.6
cd -
patch -p1<rattlesnakeos-stack-10.0.6-opengapps.patch
cd rattlesnakeos-stack
make tools && make
```

I have only tested this on my Pixel 2 (walleye), hence the specific patch for this device. PRs are welcome for other supported devices and/or a more generic patch.

Remember to add the following to your `~/.rattlesnakeos.toml` config file, following the instructions for the OpenGApps AOSP based build system (https://github.com/opengapps/aosp_build):

````toml
[[custom-patches]]
  repo = "https://github.com/corrmaan/rattlesnakeos-patches"
  patches = [
      "00001-opengapps-find-apk-for-pkg.patch",
      "00002-opengapps-walleye.patch",
      "00003-magisk_mkbootfs.patch",
      "00004-enable-volte.patch",
  ]
  
[[custom-manifest-remotes]]
  name = "opengapps"
  fetch = "https://github.com/opengapps/"
  revision = "master"

[[custom-manifest-remotes]]
  name = "gitlab"
  fetch = "https://gitlab.opengapps.org/opengapps/"
  revision = "master"

[[custom-manifest-projects]]
  path = "vendor/opengapps/build"
  name = "aosp_build"
  remote = "opengapps"

[[custom-manifest-projects]]
  path = "vendor/opengapps/sources/all"
  name = "all"
  remote = "gitlab"

[[custom-manifest-projects]]
  path = "vendor/opengapps/sources/arm"
  name = "arm"
  remote = "gitlab"

[[custom-manifest-projects]]
  path = "vendor/opengapps/sources/arm64"
  name = "arm64"
  remote = "gitlab"

[[custom-manifest-projects]]
  path = "vendor/opengapps/sources/x86"
  name = "x86"
  remote = "gitlab"

[[custom-manifest-projects]]
  path = "vendor/opengapps/sources/x86_64"
  name = "x86_64"
  remote = "gitlab"
````

## Magisk
Please see CaseyBakey's work on including Magisk in the build. The `add_magisk()` function in their build script at https://github.com/CaseyBakey/chaosp/blob/master/build.sh should be added to `templates/build_template.go` and called after `build_aosp` and before `release "${DEVICE}"` in `full_run()`.

This is implemented in a patch for `rattlesnakeos-stack` available [here](https://gist.githubusercontent.com/corrmaan/5966d62f050d0f0cf22d7af60c37aac3/raw/50da4db5386bf791feda293db913741530e956f1/rattlesnakeos-stack-10.0.6-magisk.patch "rattlesnakeos-stack-10.0.6-magisk.patch"). After cloning `rattlesnakeos-stack`, apply the patch along with the above opengapps patch and build from source:

```bash
git clone https://github.com/dan-v/rattlesnakeos-stack.git
cd rattlesnakeos-stack
git checkout 794d5887d8e05ddcffb2d061204bd726c123e52e # 10.0.6
cd -
patch -p1<rattlesnakeos-stack-10.0.6-opengapps.patch # Optional
patch -p1<rattlesnakeos-stack-10.0.6-magisk.patch
cd rattlesnakeos-stack
make tools && make
```

Note that if Magisk is built in to RattlesnakeOS using this method it can only be applied as an OTA update. Your device will not boot if a factory image with Magisk is installed. So you'll need to first build RattlesnakeOS without Magisk (but opengapps can be included), flash that factory image, then build RattlesnakeOS with Magisk (and opengapps as well if it was built in to the factory image) and apply the new OTA manually or let your device update automatically over the internet.
