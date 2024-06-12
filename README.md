# Not Maintained Anymore
---

[![GoDoc Widget]][GoDoc]



`hc` handles the underlying communication between HomeKit accessories and clients.
You can focus on implementing the business logic for your accessory, without having to worry about the protocol.

**What is HomeKit?**

[HomeKit][homekit] is a set of protocols and libraries from Apple. It is used by Apple's platforms to communicate with smart home appliances. 

HomeKit is fully integrated into iOS since iOS 8. 

<img alt="Home+.app" src="_img/home-icon.png?raw=true" width="87" />

I've developed the [Home+][home+] app to control HomeKit accessories from iPhone, iPad, and Apple Watch.
If you want to support `hc`, please purchase Home from the [App Store][home-appstore]. That would be awesome. ❤️

## Features

- Supports Go modules (requires Go 1.13)
- Full implementation of the HAP in Go
- Runs on linux and macOS

## Getting Started

        # Run the project
        make run

3. Pair with your HomeKit App of choice (e.g. [Home][home-appstore])

**Go Modules**

Make sure to set the environment variable `GO111MODULE=on`.

## Example

See [_example](_example) for a variety of examples.

**Basic switch accessory**

Create a simple on/off switch, which is accessible via IP and secured using the pin *00102003*.

```go
package main

import (
    "log"
    "github.com/brutella/hc"
    "github.com/brutella/hc/accessory"
)

func main() {
    // create an accessory
    info := accessory.Info{Name: "Lamp"}
    ac := accessory.NewSwitch(info)

    // configure the ip transport
    config := hc.Config{Pin: "00102003"}
    t, err := hc.NewIPTransport(config, ac.Accessory)
    if err != nil {
        log.Panic(err)
    }

    hc.OnTermination(func(){
        <-t.Stop()
    })

    t.Start()
}
```

You can define more specific accessory info, if you want.

```go
info := accessory.Info{
    Name: "Lamp",
    SerialNumber: "051AC-23AAM1",
    Manufacturer: "Apple",
    Model: "AB",
    FirmwareRevision: "1.0.1",
}
```

### Events

The library provides callback functions, which let you know when a clients updates a characteristic value.

```go
ac.Switch.On.OnValueRemoteUpdate(func(on bool) {
    if on == true {
        log.Println("Switch is on")
    } else {
        log.Println("Switch is off")
    }
})
```

When the switch is turned on "the analog way", you should set the state of the accessory.

```go
ac.Switch.On.SetValue(true)
```

## Multiple Accessories

When you create an IP transport, you can specify more than one accessory like this

```go
bridge := accessory.NewBridge(...)
outlet := accessory.NewOutlet(...)
lightbulb := accessory.NewColoredLightbulb(...)

hc.NewIPTransport(config, bridge.Accessory, outlet.Accessory, lightbulb.Accessory)
```

By doing so, the *bridge* accessory will become a HomeKit bridge.
The *outlet* and *lightbulb* are the bridged accessories.

When adding the accessories to HomeKit, iOS only shows the *bridge* accessory.
Once the bridge was added, the other accessories appear automatically.

HomeKit requires that every accessory has a unique id, which must not change between system restarts.
`hc` automatically assigns the ids for you based on the order in which the accessories are added to the bridge.


```go
bridge := accessory.NewBridge(accessory.Info{Name: "Bridge", ID: 1})
outlet := accessory.NewOutlet(accessory.Info{Name: "Outlet", ID: 2})
lightbulb := accessory.NewColoredLightbulb(accessory.Info{Name: "Light", ID: 3})
```

## Accessory Architecture

HomeKit uses a hierarchical architecture to define accessories, services and characeristics.
At the root level there is an accessory.
Every accessory contains services.
And every service contains characteristics.
