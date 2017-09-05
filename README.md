# fritzapi
[![NPM Version](https://img.shields.io/npm/v/fritzapi.svg)](https://www.npmjs.com/package/fritzapi)
[![Build status](https://travis-ci.org/andig/fritzapi.svg?branch=master)](https://travis-ci.org/andig/fritzapi)
[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=HWZTN5AU8LSUC)

Home automation node API for Fritz!Box, Fritz!DECT and FRITZ!Powerline devices. 

[homebridge-fritz](https://github.com/andig/homebridge-fritz) and [pimatic-fritz](https://github.com/andig/pimatic-fritz) are built on fritzapi.

## Functions

### General functions

- Get the session ID `getSessionID`
- Get device list as XML `getDeviceListInfo` >FritzOS 6.10
- Get device list `getDeviceList` >FritzOS 6.10
- Get device `getDevice` >FritzOS 6.10
- Get temperature `getTemperature` - polyfill
- Get presence `getPresence` - polyfill

**Note**

`getTemperature` is not available on the FRITZ!Powerline 546E WLAN set and will always return `NaN`.

### Fritz!DECT 200 and 210 outlet functions (includes FRITZ!Powerline 546E)

- Get list `getSwitchList`
- Get state `getSwitchState`
- Set on `setSwitchOn`
- Set off `setSwitchOff`
- Get power `getSwitchPower`
- Get energy `getSwitchEnergy`
- Get presence status `getSwitchPresence`
- Get name `getSwitchName`

For controlling AVM Fritz!DECT 200 devices the actuator identification number (AIN) is needed. The AIN can be obtained using `getSwitchList` which returns a list of AINs or the more general `getDeviceList` function which returns a verbose device list structure as JSON.

`getTemperature` is not available for switch groups that can be created through the Fritz!Box user interface.

### Fritz!DECT 100 functions

The Fritz!DECT 100 DECT repeater AIN does only appear in the `getDeviceList` output. It supports retrieving the repeaters temperature.

### Fritz!DECT 300 and CometDECT thermostat functions

Thermostat functions are only available as of FritzOS 6.36

- Get list `getThermostatList` - polyfill
- Set target temperature `setTempTarget`, supports 'ON'/'OFF' to enable/disable thermostat
- Get target temperature `getTempTarget`
- Get comfort temperature `getTempComfort`
- Get night temperature `getTempNight`
- Get battery charge status `getBatteryCharge`


### WLAN functions

- Get the guest wlan settings `getGuestWlan`
- Set the guest wlan `setGuestWlan`

**Note**

`getGuestWlan` returns a structure containing all wifi settings found in the Fritz!Box UI. The `setGuestWlan` function accepts either a settings structure such as this or a single boolean value.

All functions have been tested on FritzOS 6.20/6.36/6.51 using the Fritz!Box 7390 and on FritzOS 6.50 on Fritz!Box 6490. The WLAN functions may be less stable.


## Installation

```bash
npm install fritzapi
```


## Usage

### Object-oriented interface

The object-oriented interface is the recommended way of using fritzapi.

Get the list of switch AINs using a customer Fritz!Box address:
```js
var Fritz = require('fritzapi').Fritz;

var f = new Fritz("user", "password", "192.168.178.1");

f.getSwitchList().then(function(ains){
  console.log(f.getSID());
  console.log(ains);
});
```

### Functional interface

Get the session ID using default Fritz!Box address (http://fritz.box):
```js
var fritz = require('fritzapi');

fritz.getSessionID("user", "password").then(function(sid) {
    console.log(sid);
});
```

Get the list of switch AINs using a custom Fritz!Box address with self-signed certificate:
```js
fritz.getSessionID("user", "password", {
  url: "192.168.178.1",
  strictSSL: false         // workaround DEPTH_ZERO_SELF_SIGNED_CERT SSL error
}).then(function(sid) {
  console.log(sid);

  // note that the options/url need be carries through every single api call
  fritz.getSwitchList(sid, options).then(function(ains){
    console.log(ains);
  });
});
```

**Note** The functional interface may be deprecated in a future version of the library.


## Device details

For sake of reference bolow is the output of `getDeviceList` as returned for the various Fritz devices I'm having around. These definitions remain cached by the Fritz!Box even if the device is no longer connected. The device presence is indicated by the `present` attribute.

### Powerline

    { identifier: '34:31:C4:DB:F6:C7',
      id: '20000',
      functionbitmask: '640',
      fwversion: '06.20',
      manufacturer: 'AVM',
      productname: 'FRITZ!Powerline 546E',
      present: '0',
      name: 'FRITZ!Powerline 546E',
      switch: { state: '', mode: '', lock: '' },
      powermeter: { power: '', energy: '' } }

### Outlets

    { identifier: '08761 0103568',
      id: '16',
      functionbitmask: '640',
      fwversion: '03.59',
      manufacturer: 'AVM',
      productname: 'FRITZ!DECT 200',
      present: '0',
      name: 'FRITZ!DECT 200 #1',
      switch: { state: '', mode: '', lock: '' },
      powermeter: { power: '', energy: '' } }

    { identifier: '11657 0031825',    
      id: '18',
      functionbitmask: '640',
      fwversion: '03.67',
      manufacturer: 'AVM',
      productname: 'FRITZ!DECT 210',
      present: '0',
      name: 'FRITZ!DECT 210 #3',
      switch: { state: '', mode: '', lock: '' },
      powermeter: { power: '', energy: '' } }

### DECT Repeater

    { identifier: '11657 0002527',
      id: '20',
      functionbitmask: '1024',
      fwversion: '03.64',
      manufacturer: 'AVM',
      productname: 'FRITZ!DECT Repeater 100',
      present: '0',
      name: 'FRITZ!DECT Rep 100 #5' }

### Thermostats

    { identifier: '10971 0195784',
      id: '17',
      functionbitmask: '320',
      fwversion: '03.66',
      manufacturer: 'AVM',
      productname: 'Comet DECT',
      present: '0',
      name: 'Comet DECT',
      temperature: { celsius: '', offset: '' },
      hkr: { tist: '', tsoll: '', absenk: '', komfort: '' } }


## AHA HTTP Interface Documentation

http://avm.de/fileadmin/user_upload/Global/Service/Schnittstellen/AHA-HTTP-Interface.pdf


## Acknowledgements

Thanks to:

* AVM for providing free test devices and the AHA-HTTP interface document
* EUROtronic Technology GmbH for providing free CometDECT thermostat sample
* nischelwitzer for the basic js implementation (https://github.com/nischelwitzer/smartfritz)
* steffen.timm for the basic communication function
* thk4711 for the FRITZ!DECT 200 codes
