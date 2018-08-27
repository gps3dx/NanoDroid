# Deodex services.jar

Here you can find instructions on how to manually deodex `services.jar` as preparation for Signature Spoofing support.

If you want to check beforehand (or afterwards), if your ROM supports Signature Spoofing, use this [tool](https://f-droid.org/de/packages/lanchon.sigspoof.checker/).

## VDEX

If you can see `/system/framework/oat/[arch]/services.vdex`, where [arch] is the device architecture (arm, arm64, x86 or x86_64), you should follow theese instructions.

the instructions are basically simple, though plenty commands:

* connect phone with PC while in TWRP, mount `/system` **read-write**
* get latest `vdexExtractor` (best self-compiled from git)
  * [vdexExtractor Sources](https://github.com/anestisb/vdexExtractor)
* zip/unzip utilities for commandline (or some UI tool)
* common sense on using commandline (or some UI tool instead)

```
adb pull /system/framework framework
cp framework/services.jar services.jar-backup
```

now deodex `services.vdex`:

```
vdexExtractor -i framework/oat/[arch]/services.vdex --ignore-crc-error
```

this will create the following file:
* framework/oat/[arch]/services.apk_classes.dex

if it's properly been created rename it to classes.dex and add it to `services.jar`

```
mv framework/oat/[arch]/services.apk_classes.dex classes.dex
zip -d framework/services.jar classes.dex
zip -j framework/services.jar classes.dex
```

next install the new `services.jar` to device:

```
adb push framework/services.jar /system/framework
adb shell
chmod 0644 /system/framework/services.jar
chown root:root /system/framework/services.jar
```

unmount `/system` and flash the NanoDroid-Patcher.

If something goes wrong you still have the unpatched `services.jar`, as we created a copy named `services.jar-backup`.

## ODEX

If you can see `/system/framework/oat/[arch]/services.odex`, where [arch] is the device architecture (arm, arm64, x86 or x86_64), you should follow theese instructions.

the instructions are basically simple, though plenty commands:

* connect phone with PC while in TWRP, mount `/system` **read-write**
* get latest `baksmali.jar` and `smali.jar` (2.2.4+, best self-built from git)
  * [smali Sources](https://github.com/JesusFreke/smali)
  * my personal builds [baksmali](http://nanolx.org/apk/baksmali-2.2.4-86963ccb-fat.jar) / [smali](http://nanolx.org/apk/baksmali-2.2.4-86963ccb-fat.jar)
* zip/unzip utilities for commandline (or some UI tool)
* common sense on using commandline (or some UI tool instead)

```
adb pull /system/framework framework
cp framework/services.jar services.jar-backup
```

check if `framework/services.jar` contains a `classes.dex` (open as zip file), if so:

```
mkdir services-old
unzip -d services-old framework/services.jar
java -jar baksmali.jar x services-old/classes.dex -b framework/core-oj.jar -o services-new
java -jar smali.jar a services-new -o classes.dex
```

if ***not***

```
java -jar baksmali.jar x framework/oat/[arch]/services.odex -b framework/core-oj.jar -o services-new
java -jar smali.jar a services-new -o classes.dex
```

in any case, if a new classes.dex was successfully created in the services-new directory, re-package it into the services.jar we previously pulled:

```
zip -d framework/services.jar classes.dex
zip -j framework/services.jar classes.dex
```

next install the new `services.jar` to device:

```
adb push framework/services.jar /system/framework
adb shell
chmod 0644 /system/framework/services.jar
chown root:root /system/framework/services.jar
```

unmount `/system` and flash the NanoDroid-Patcher.

If something goes wrong you still have the unpatched `services.jar`, as we created a copy named `services.jar-backup`.