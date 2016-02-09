MMcontrol provides an easy way to control Mitsubishi heat pumps that use the MAC-558 WiFi Adaptor (sold in New Zealand and Australia).

## Features

MMcontrol provides the following features:
* setting the power state 
* setting the operation mode (cool, heat, dry, fan, auto)
* setting speed of the fans
* setting the target temperature
* setting the airflow direction (horizontal and vertical)
* reading current state of the unit (power mode, standby mode, operation mode, fan speed, room temperature and target temperature)
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
```
### Query capabilities of unit 0
```javascript
controller.getCapabilities(0, function (err, capabilities) {
    if (err) {
        console.log("couldn't get the capabilities: " + err);
    }
    console.log(JSON.stringify(capabilities, null, 1));
});
```
output:
```JSON
    {
     "action": [
      "mode",
      "fan",
      "power",
      "temperature"
     ],
     "mode": [
      "heat",
      "dry",
      "cool",
      "fan",
      "auto"
     ],
     "power": [
      "on",
      "off"
     ],
     "fan": [
      "1",
      "2",
      "3"
     ],
     "airDirH": [],
     "airDirV": [
      "auto"
     ]
    }
```
### Query current state of unit 0
```javascript
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
     "mode": "cool",
     "automode": "",
     "standby": "off",
     "fanSpeed": "2",
     "power": "on",
     "setTemperature": 23,
     "roomTemperature": 24,
     "airDirV": "",
     "airDirH": ""
    }
```
### Set the mode of unit 0 to 'cool'
```javascript
controller.setMode(0, 'cool', function (err) {
    if (err) {
        console.log("couldn't set the mode: " + err)
    }
    console.log("mode set");
});

```
## API

#### MMcontrol(params)
Constructor, sets initial values
 * params object that contains parameters used to initialise the object:
 
| parameter | type |definition |   | default value |
| ----------|----|-------------------|---|---------------|
| url | *string* |address the heat pump API resides at | optional | https://api.melview.net/ |
| username | *string* | username (email address) used at the Mitsubishi site/in the app | required |  |
| password | *string* | password used at the Mitsubishi site/in the app | required | |
| userAgent | *string* | user agent that should be used for all requests | restify default | |
| log | *object* | bunyan log object, MMcontrol will log using it (module=MMcontrol, level: trace) | optional | |
| minRefresh | *integer* | duration (in seconds) MMcontrol will wait before querying the API again to refresh the state of the heat pump ('set' requests are send immiedietaly) | optional | 60 |
| tmpDir | *string* | directory used to store temporary files (cookies, capabilities and state if persistence is enabled) | optional | /tmp |
| persistence | *bool* | if MMcontrol should store cookies, capabilities of the heat pump(s) and the state(s) in a file for re-use after process terminates | optional | true |

a *TypeError* exception is thrown if required parameters are missing.

#### connect (reuse, callback)
Builds a connection with the API or loads details of the previous one from a file. If a file can't be loaded a new connection will be build.
* reuse (*bool*) - should the previous state be loaded.
* callback (*function* (error)) - called on completion, if there was a problem 'error' contains a *string*, *null* otherwise

#### getUnitList (callback)
Returns an *array* with unit names. Index of the array can be used to identify the heat pump units in subsequent set/get calls.
* callback (*function* (error, unitList)) - called on completion, if there was a problem 'error' contains a *string*, *null* otherwise, unitList contains an *array* of unit names

#### getCapabilities (unitid, callback)
Returns an *object* with capabilities of the unit, based on the information returned by the remote unit API.
* unitid (*integer*) - the id of the unit (as returned by *getUnitList*)
* callback (*function* (error, capabilities)) - called on completion, if there was a problem 'error' contains a *string*, *null* otherwise, capabilities contains the capabilities of the  unit:

| parameter | type | definition |
|---|---|---|
| action | array | set of commands that the unit responds to |
| mode | array | list of supported modes |
| power | array | list of supported power states (always 'on' and 'off') |
| fan | array | list of supported fan speeds |
| airDirH | array | list of supported horizontal airflow directions |
| airDirV | array | list of supported vertical airflow directions |

#### getCurrentState (unitid, callback)
Returns an *object* with the current state of the heat pump unit
* unitid (*integer*) - the id of the unit (as returned by *getUnitList*)
* callback (*function* (error, state)) - called on completion, if there was a problem 'error' contains a *string*, *null* otherwise, state contains the current state of the unit:

| parameter | type | definition |
|---|---|---|
| mode | *string* | currently set mode ('cool', 'heat', 'dry', 'fan', 'auto') |
| automode | *string* | if mode is set to 'auto' contains actual operating mode, empty string otherwise |
| standby | *string* | if the unit is in standby mode ('on', 'off') |
| fanSpeed | *string* | currently set fan speed |
| power | *string* | current power mode ('on', 'off') |
| setTemperature | *float* | currently set target temperature |
| roomTemperature | *float* | currently reported room temperature |
| airDirH | *string* | currently set horizontal air direction, empty string if the capability is not supported |
| airDirV | *string* | currently set vertical air direction, empty string if the capability is not supported |

If the power is 'off' the reported values are the last active ones from before the unit was switched off. Current roomTemperature is always reported (even when the unit is off), the reported temperature is always rounded to the nearest *integer* (this is a limitation of the API).

#### getCurrentStateRaw (unitid, callback)
Returns an *object* with the current state of the heat pump unit as returned by the API (without being normalised with values from the model file)
* unitid (*integer*) - the id of the unit (as returned by *getUnitList*)
* callback (*function* (error, state)) - called on completion, if there was a problem 'error' contains a *string*, *null* otherwise, state contains the current state of the unit.

#### setPower (unitid, state, callback)
Turns the unit off or on.
* unitid (*integer*) - the id of the unit (as returned by *getUnitList*)
* state (*string*) - 'on' or 'off' (as defined in the model file)
* callback (*function* (error) - called on completion, if there was a problem 'error' contains a *string*, *null* otherwise

#### setTemperature (unitid, temperature, callback)
Sets the target temperature. Only works in modes that support temperature ('cool', 'heat', 'auto'), in others quietly returns. The value of the temperature is adjusted to fit within the range allowed by the unit.
* unitid (*integer*) - the id of the unit (as returned by *getUnitList*)
* temperature (*float*) - the temperature to set (should be set to half-degree resolution)
* callback (*function* (error)) - called on completion, if there was a problem 'error' contains a *string*, *null* otherwise

#### setMode(unitid, mode, callback)
Sets the mode of operation ('cool', 'heat', 'dry', 'fan', 'auto'). Adjusts target temperature to sit within the range allowed by the unit.
* unitid (*integer*) - the id of the unit (as returned by *getUnitList*)
* mode (*string*) - one of 'cool', 'heat', 'dry', 'fan', 'auto'
* callback (*function* (error)) - called on completion, if there was a problem 'error' contains a *string*, *null* otherwise

#### setFanSpeed(unitid, fanSpeed, callback)
Sets the speed of the fan.
* unitid (*integer*) - the id of the unit (as returned by *getUnitList*)
* fanSpeed (*string*) - speed to set (1-5, auto)
* callback (*function* (error)) - called on completion, if there was a problem 'error' contains a *string*, *null* otherwise

#### setAirDirH(unitid, dir, callback)
Sets the horizontal direction of the airflow
* unitid (*integer*) - the id of the unit (as returned by *getUnitList*)
* dir (*string*) - direction (0-5, auto, swing)
* callback (*function* (error)) - called on completion, if there was a problem 'error' contains a *string*, *null* otherwise

#### setAirDirV(unitid, dir, callback)
Sets the vertical direction of the airflow
* unitid (*integer*) - the id of the unit (as returned by *getUnitList*)
* dir (*string*) - direction (0-5, auto, swing)
* callback (*function* (error)) - called on completion, if there was a problem 'error' contains a *string*, *null* otherwise

## Capabilities
MMcontrol detects the capabilities of the heat pump and limits the commands to what the unit reports as enabled.

The following functions are always enabled:
* mode:
 - heat
 - cool
 - fan
* setting temperature
* turning power on/off
* setting fan speed
 - limited to the speeds reported by the unit

The following functions are only enabled if the unit reports back the capability
* mode:
 - auto
 - dry
* setting fan speed to auto
* changing vertical direction of the airflow
 - directions 1-5 are enabled by default 
 - auto and swing - if enabled by the unit
* changing horizontal direction of the airflow
 - directions 1-5, swing and auto are enabled by default
  
## Request optimisations

MMcontrol tries to limit the number of requests used to query the current state of the heat pump ('get' commands, i.e. not changing the state of the unit). By default, the state is cached for 60 seconds (this value can be changed by passing the *minRefresh* parameter to the constructor). Each set-type command is send immediately and also refreshes the current state of the unit. 

## Limitations

So far the module has been tested only with the following heat pumps:
* ducted (PEAD-RPxx)

## Troubleshooting 
If for some reason the module refuses to work correctly please remove the state file (usually /tmp/state.txt) or *connect* with the reuse option set to *false*.

## Disclaimer

This software is not affiliated with Mitsubishi in any way, nor am I. Use at your own risk.