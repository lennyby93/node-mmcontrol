MMcontrol provides an easy way to control Mitsubishi heat pumps that use the MAC-558 WiFi Adaptor (sold in New Zealand and Australia).

## Features

MMcontrol provides the following features:
* setting the power state 
* setting the operation mode (cool, heat, dry, fan, auto)
* setting speed of the fans
* setting target temperature
* reading current state of the unit (power mode, standby mode, operation mode, fan speed, room temperature and target temperature )
* supports multiple units under a single account
* session persistence between subsequent calls
* integrates with bunyan logger

## Install
```bash
npm install mmcontrol
```

## Setup

In order for the module to work you must set up an account with Mitsubishi (either using the browser or using the phone app). Once you've added your unit please make sure that the "Half degree temperature" is enabled.

## Example

```javascript
var MMcontrol = require('mmcontrol');

var controller = new MMcontrol({
    'username': 'username@domain.com',
    'password': 'mypassword'
});
```
### Establish a new connection
```javascript
controller.connect(true, function (err) {
    if (err) {
        console.log("couldn't connect: " + err);
    } else {
        console.log("connection established");
    });

### Query current state of unit 0
controller.getCurrentState(0, function (err, state) {
    if (err) {
        console.log("couldn't get the current state: " + err);
    }
    console.log(JSON.stringify(state, null, 1));
});
```
output:
```JSON
    {
     "mode": "auto",
     "automode": "cool",
     "standby": "off",
     "fanSpeed": "1",
     "power": "on",
     "setTemperature": 22,
     "roomTemperature": 23
    }
```
### Set the mode of unit 0 to 'cool'
```javascript
controller.setMode(0, 'cool', function (err) {
    if (err) {
        console.log("couldn't set the mode: " + err)
    }
    console.log("mode set")
});

```
## API

#### MMcontrol(params)
Constructor, sets initial values
 * params object that contains parameters used to initalise the object:
 
| parameter | type |definition |   | default value |
| ----------|----|-------------------|---|---------------|
| url | *string* |address the heat pump API resides at | optional | https://api.melview.net/ |
| username | *string* | username (email address) used at the Mitsubishi site/in the app | required |  |
| password | *string* | password used at the Mitsubishi site/in the app | required | |
| userAgent | *string* | user agent that should be used for all requests | restify default | |
| log | *object* | bunyan log object, MMcontrol will log using it (module=MMcontrol, level: trace) | optional | |
| minRefresh | *integer* | duration (in seconds) MMcontrol will wait before querying the API again to refreash the state of the heat pump ('set' requests are send immiedietaly) | optional | 60 |
| tmpDir | *string* | directory used to store temporary files (cookies, capabilites and state if persistance is enabled) | optional | /tmp |
| persistance | *bool* | if MMcontrol should store cookies, capabilities of the heat pump(s) and the state(s) in a file for re-use after process terminates | optional | true |

a *TypeError* exception is thrown if required parameters are missing.

#### connect (reuse, callback)
Builds a connection with the API or loads details of the previous one from a file. If a file can't be loaded a new connection will be build.
* reuse (*bool*) - should the previous state be loaded.
* callback (*function* (error)) - called on completion, if there was a problem 'error' contains a *string*, *null* otherwise

#### getUnitList (callback)
Returns an *array* with unit names. Index of the array can be used to identify the heat pump units in subsequent set/get calls
* callback (*function* (error, unitList)) - called on completion, if there was a problem 'error' contains a *string*, *null* otherwise, unitList contains an *array* of unit names

#### getCurrentState (unitid, callback)
Returns an *object* with the current state of the heat pump unit
* unitid (*integer*) - the id of the unit (as returned by *getUnitList*)
* callback (*function* (error, state)) - called on completion, if there ws a problem 'error' contains a *string*, *null* otherwise, state contains the current state of the unit:

| parameter | type | definition |
|---|---|---|
| mode | *string* | currently set mode ('cool', 'heat', 'dry', 'fan', 'auto') |
| automode | *string* | if mode is set to 'auto' contains actual operating mode |
| standby | *string* | if the unit is in standby mode ('on', 'off') |
| fanSpeed | *string* | currently set fan speed |
| power | *string* | current power mode ('on', 'off') |
| setTemperature | *float* | currently set target temperature |
| roomTemperature | *float* | currently reported room temperature |

If the power is 'off' the reported values are the last active ones from before the unit was switched off. Current roomTemperature is always reported (even when the unit is off), the reported temperature is always rounded to the nearest *integer*.

#### getCurrentStateRaw (unitid, callback)
Returns an *object* with the current state of the heat pump unit as returned by the API (without being normalised with values from the model file)
* unitid (*integer*) - the id of the unit (as returned by *getUnitList*)
* callback (*function* (error, state)) - called on completion, if there ws a problem 'error' contains a *string*, *null* otherwise, state contains the current state of the unit.

#### setPower (unitid, state, callback)
Turns the unit off or on.
* unitid (*integer*) - the id of the unit (as returned by *getUnitList*)
* state (*string*) - 'on' or 'off' (as defined in the model file)
* callback (*function* (error) - called on completion, if there ws a problem 'error' contains a *string*, *null* otherwise

#### setTemperature (unitid, temperature, callback)
Sets the target temperature. Only works in modes that support temperature ('cool', 'heat', 'auto'), in others quietly returns. The value of the temperature is adjusted to fit within the range allowed by the unit.
* unitid (*integer*) - the id of the unit (as returned by *getUnitList*)
* temperature (*float*) - the temperature to set (should be set to half-degree resolution)
* callback (*function* (error)) - called on completion, if there ws a problem 'error' contains a *string*, *null* otherwise

#### setMode(unitid, mode, callback)
Sets the mode of operation ('cool', 'heat', 'dry', 'fan', 'auto'). Adjusts target temperature to sit within the range allowed by the unit.
* unitid (*integer*) - the id of the unit (as returned by *getUnitList*)
* mode (*string*) - one of 'cool', 'heat', 'dry', 'fan', 'auto'
* callback (*function* (error)) - called on completion, if there ws a problem 'error' contains a *string*, *null* otherwise

#### setFanSpeed(unitid, fanSpeed, callback)
Sets the speed of the fan.
* unitid (*integer*) - the id of the unit (as returned by *getUnitList*)
* fanSpeed (*string*) - speed to set (as defined in the model file)
* callback (*function* (error)) - called on completion, if there ws a problem 'error' contains a *string*, *null* otherwise

## Model definitions

The commands and values used to control a heat pump unit are defined in model file. The file contains the mappings between then internal and human-readable values. A file for type-3 looks like this:
```JSON
{
    "prefix": {
        "mode": "MD",
        "fan": "FS",
        "power": "PW",
        "temperature": "TS"
    },
    "mode": {
        "heat": "1",        
        "dry": "2",        
        "cool": "3",
        "fan": "7",
        "auto": "8"
    },
    "power": {
        "on": "1",
        "off": "0"
    },
    "fan": {
        "1": "2",
        "2": "3",
        "3": "5"
    }
}
```
* prefix - states the prefix that should be used for the various 'set' commands
* mode - defines the modes heat pump unit understands
* power - defines the power stats for the heat pump
* fan - defines the fan speeds

## Limitations

So far the module has been tested only with the following heat pumps:
* ducted (PEAD-RPxx)

## Disclaimer

This software is not affiliated with Mitsubishi in any way, nor am I. Use at your own risk.