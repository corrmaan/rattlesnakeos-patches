# rattlesnakeos-patches
My patches for Rattlesnake OS.

## opengapps
Note that building opengapps with Rattlesnake OS will require a larger EC2 volume size. By default it's set to 200 GB, but can be increased by editing the value of the `VolumeSize` key in `templates/lambda_template.go`. 300 worked for me.

As well, opengapps' aosp_build requires `zlib`. I created a script to install it, available at https://github.com/corrmaan/rattlesnakeos-scripts.

I have only tested this on my Pixel 2 (walleye), hence the specific patch for this device. PRs are welcome for other supported devices and/or a more generic patch.

## Magisk
Please see CaseyBakey's work on including Magisk in the build. The `add_magisk()` function in their build script at https://github.com/CaseyBakey/chaosp/blob/master/build.sh should be added to `templates/build_template.go` and called after `build_aosp` and before `release "${DEVICE}"` in `full_run()`.
