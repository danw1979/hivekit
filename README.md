# HiveKit [![Circle CI](https://circleci.com/gh/njpatel/hivekit.svg?style=svg)](https://circleci.com/gh/njpatel/hivekit)

**NOTE**

**The Hive Home service has moved onto a new API for their devices, HiveKit will no longer work.**

**Please see [this issue](https://github.com/njpatel/hivekit/issues/7) for more details. Sorry for the inconvenience.**

**/NOTE**


HiveKit is a [HomeKit](https://developer.apple.com/homekit/) bridge to make the [Hive Active Heating](https://hivehome.com) system available as HomeKit accessories. This allows you to use your Hive with your iPhone, Apple Watch, and Siri.

HiveKit is based on the excellent [HomeControl](https://github.com/brutella/hc) by [@brutella](https://twitter.com/brutella).

![](https://i.imgur.com/GOQYsj1.gif)

## Overview
HiveKit is a daemon that registers itself as a HomeKit *bridge* with three *accessories*:

 * **Heating** *thermostat*
 * **Heating Boost** *switch*
 * **Hot Water** *switch*
 
These accessories expose some of the functionality of your Hive device to HomeKit. Once a connection is set up, you can view the status of these accessories & manipulate them via any HomeKit-compatible application and/or Siri, on your iPhone/iPad/Apple Watch.

HiveKit connects to the [hivehome.com](https://hivehome.com) web service to read & manipulate the Hive components in your home. Every installation of the Hive should come with a login to the aformentioned service (usually and email + password), and the same login details are used for the Hive application on iOS/Android.

**Note:** The way HomeKit works means that it has a very strict world-view of home automation accessories, and therefore HiveKit does it's best to fit the Hive device and it's capabilities into this world-view. This means that, while the most important features are available, not all the features of the Hive device have been mapped as yet. As HomeKit & HiveKit mature, these will be made available.

## Features
 * View & set the current & target temperature on the Hive via any HomeKit-enabled app and/or Siri
 * Boost the heating (and manipulate the boost) via an app and/or Siri
 * Boost the hot water (and manipulate the boost) via an app and/or Siri
 
## HomeKit App v. HomeKit Bridge
The terminology and architecture around HomeKit can be confusing and therefore hard to figure out how things fit together. 

**HiveKit** is a *HomeKit Bridge* daemon, it's meant to be run on your computer and allowed to go about it's business. It's run from the command-line and has no GUI itself.

**HomeKit** is the database/register on your phone that comes as part of iOS 8.0 and above. Again, it's lacks any UI itself, and is rather a service that runs in the background waiting for an *HomeKit app* to register some accessories with it. Once accessories have been registered, HomeKit springs to life and makes them available to Siri and relays commands back to the HomeKit App. Without a HomeKit App, HomeKit itself is useless.

A **HomeKit App** is an app downloaded from the App Store that can hook devices/bridges that support HomeKit on your home network to the HomeKit database on your phone. Without such an app, neither HomeKit or HiveKit are of any use. Some example apps are:

 * [Home](https://itunes.apple.com/app/id995994352)
 * [Devices](https://itunes.apple.com/gb/app/devices/id966877433?mt=8)
 
HiveKit is developed and tested with [Home](https://itunes.apple.com/app/id995994352). The developer of Home also produced the [open-source library](https://github.com/brutella/hc) that HiveKit uses to present itself as a HomeKit bridge. Supporting Home inadvertedly supports HiveKit :)
 
 
## Getting Started
HiveKit is written in [Go](http://golang.org/doc/install) and you will need a working Go 1.5 installation and workspace to build and run HiveKit.

**Clone & Build HiveKit**
        
```
git clone github.com/njpatel/hivekit.git
hivekit        
make dist
```
The `hivekit` binary will now be available at `./bin/hivekit`

## Running hivekit
```
$ ./bin/hivekit --help
Usage of ./bin/hivekit:
  -boost-duration int
        Duration (minutes) to boost heating (default 60)
  -boost-water int
        Duration (minutes) to boost hot water (default 60)
  -password string
        Hive Home web service password
  -pin string
        The HomeKit accessory pin (8 numeric chars)
  -username string
        Hive Home web service username (usually an email address)
  -verbose
        Enable verbose logging
```

Example:

```
./bin/hivekit -username me@example.com -password FooBar2000 -pin 80808080
```
**Note:** The PIN is essential and there is no default. It's required later in the setup procedure on the phone.

## Pairing
Once HiveKit is running, it has to be paired with an HomeKit App (as discussed above). Instructions for pairing with [Home](https://itunes.apple.com/app/id995994352) are shown below:

![Pairing Instructions](https://i.imgur.com/oZPU27K.gif)

## Siri Commands
These will all work with Siri once HiveKit is paired up:

**Hive Temperatures**

 * *What's the temperature inside the house?*
 * *What's the temperature inside?*
 * *What's the temperature of the heating?* (the temperature Hive is currently trying to achieve)
 * *Set the heating to 24 degrees* (updates the Hive's target temperature)
 
**Heating Boost**

The Hive has a nice feature where you can boost the heating for a bit if it's getting chilly. There isn't a 1:1 mapping to this feature in HomeKit, so I've exposed it as a switch. You control how long the boost lasts for via the `hivekit` command-line options.

 * *Is the heating boost on?*
 * *Turn on the heating boost*

**Hot Water Boost**

Exactly like the Heating Boost, Hot Water doesn't map well so instead it's exposed as a switch:

* *Is the Hot Water on?*
* *Turn on the hot water*

**Note:** When you 'turn off' the Heating Boost or Hot Water, the Hive is set back to the scheduled mode for heating/hot water, respectively.


## Unsupported Features

#### HomeKit
Heating/Cooling modes aren't supported. HomeKit seems to be geared for A/C and so these features don't map well to the thermostats in the UK very well. To this end, any Heat/Cool Mode options in the HomeKit App of your choice will be ignored.

#### Hive
The Hive service is complex in the way it supports schedules, frost-protect, holiday modes, schedule forwarding, etc. These features do not map well to HomeKit and therefore have been left out of the first version of HiveKit. If a good mapping can be thought up, they'll be supported in the future. Help appreciated!


## TroubleShooting
* **HiveKit doesn't show up when searching for accessories**
  * HomeKit is incredibly picky about the Bonjour connections across access points etc. For pairing, try to be on the same WiFi network as the phone.
  * Again, due to HomeKit being difficult, it sometimes helps to start HiveKit during the searching for accessories. Sometimes it helps to start it before. \_(ツ)_/¯
* **I can't log into the hivehome.com service**
  * Run `hivekit` with `-verbose` and see if any errors are printed. If everything seems right and it still doesn't work, open up an `issue`.
* **I get IPv6 errors**
  * These are fine to ignore, I'll work on silencing them from the underlying library in the future.
* **It keeps disconnecting**
  * I'm working on this - due to the way HomeKit works the issue could be from iOS, from the HomeKit App you've chosen, or HomeControl/HiveKit itself. Currently even officially supported accessories can flicker in and out of existance according to HomeKit, so it's just something to live with for now. My plan is to at least detect it and do a soft restart of the hub control in HiveKit.
  

## Contact
Neil Jagdish Patel

[Github](https://github.com/njpatel) [Twitter](https://twitter.com/njpatel) [Medium](https://medium.com/@njpatel)
