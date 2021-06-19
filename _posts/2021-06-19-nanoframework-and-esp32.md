---
layout: default
title: "Notes on using esp32 with nanoframework"
date: 2021-06-19
categories: iot microcontrollers esp32
tags: iot microcontrollers esp32
---

## [Draft] Getting started with nanoframework on esp32

### Setup local dev environment & troubleshooting

Following the docs below, some additional notes, using a ESP32-DevKitC.

- holding both BOOT and EN will reset board.
- holding BOOT will allow flash to happen & connection.

Installing VS Extension may not work 1st time - the board may not be picked up immediately. To resolve this I have proceeded with the next step to `install global nanoff tool`. Then run flash command (holding BOOT while command is running), `nanoff --serialport COM3 --stable --target ESP32_WROOM_32 --update` (make sure to target the actual COM port in use on your machine). Now device will show in Device Explorer (windows -> other -> .)

Finding out which COM port is in use can be done by the following

- windows-key -> device manager -> drill into "Ports (COM & LPT)" -> COM1 likely listed and some other port. The "other" port is the one you are using.

Nuget packages may be out of date/unavailable, I found it is best to enable preview/pre-releases in nuget package manager and go with the latest. That said, the latest may have some issues when deploying, an erorr I had was that the required version of mscorlib could not be found.:

```md
Couldn't find a valid native assembly required by mscorlib `v1.10.5.0`
```

The device capabilities (shown via device-explorer) showed:

```md
Native Assemblies:
  mscorlib v100.5.0.6, checksum 0x7B586F51
  etc.
```

Quite confusing, and these are not the same thing - we don't need both to have the same v, just compatible ones. It seems there's a difference between what the board gets flashed with via nanoff and what the solutions package requires. The fix was to view the pre-release packages and make sure they had the same version number of the missing package.

I ended up walking back the solutions mscorlib package version till `1.10.3-preview.7`. This package explicitly notes in it's description:

```md
This package includes the nanoFramework.CoreLibrary assembly for nanoFramework C# projects.
This package requires a target with mscorlib v100.5.0.6.
In case you don't need the System.Reflection API there is another NuGet package without this API.
```

The crucial part being `mscorlib v100.5.0.6` -> this matches what the device is flashed with. Other versions either didn't note the version or had a different one. I assume errors in the release pipeline on nanoframworks part at this point, it is pre-release after all.

### References

- <https://docs.nanoframework.net/index.html>

[Home]( {{ site.url }})
