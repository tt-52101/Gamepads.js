# HTML5 Gamepad API Enhancements

A simple JavaScript module for tracking Gamepads and events pertaining to their usage.

Kindly hosted by jsDelivr: https://cdn.jsdelivr.net/gh/FThompson/Gamepads.js@1.0.1/gamepads.js

```html
<script src='https://cdn.jsdelivr.net/gh/FThompson/Gamepads.js@1.0.1/gamepads.js' type='text/javascript'>
```

### Why use this library instead of the built-in Gamepad API?

The [existing Gamepad standard](https://developer.mozilla.org/en-US/docs/Web/API/Gamepad_API/Using_the_Gamepad_API) lacks support for button/joystick events and `gamepadconnected`/`gamepaddisconnected` events do not work consistently across browsers. This module seeks to offer a standard event-handling implementation across multiple browsers.

## Example ##

```javascript
gamepads.addEventListener('connect', (gamepad) => {
    console.log('gamepad connected')
    gamepad.addEventListener('buttonpress', (i) => console.log(`${i} pressed`))
    gamepad.addEventListener('joystickmove', (is, values) => console.log(`${is} is ${values}`), [0, 1])
})
gamepads.addEventListener('disconnect', (gamepad) => {
    console.log('gamepad disconnected')
})
gamepads.start()
```

# Usage #

## `GamepadHandler` object ##

```javascript
gamepads
```

A `GamepadHandler` singleton is exposed as `gamepads`. This object takes care of polling the HTML5 API for gamepads and tracking which gamepads are currently connected. Call `gamepads.start()` to begin polling.

### Methods ###

```javascript
gamepads.addEventListener(event, callback)
```

Adds an event listener to the page's gamepad handler.

```javascript
gamepads.removeEventListener(event, callback)
```

Removes an event listener from the page's gamepad handler.

#### Events ####

* `event 'connect': callback(gamepad)`

   Fires when a gamepad is connected or accessed for the first time (if already physically connected).

* `event 'disconnect': callback(gamepad)`

   Fires when a gamepad is disconnected.

```javascript
gamepads.start()
```

Begins polling and updating available Gamepads at the screen frame rate.

```javascript
gamepads.poll()
```

Polls and updates all available Gamepads. You may call this manually (via `setInterval` for example) if you need the module to update gamepads at a different rate than the screen frame rate.

```javascript
gamepads.stop()
```

Pauses polling and updating of gamepads. May be resumed by `start()`.

### Properties ###

```javascript
gamepads.gamepads
```

The map of all connected `Gamepad` objects `{ index: gamepad }`.

## `Gamepad` object ##

Retrieve `Gamepad` instances via `GamepadHandler`'s `connect`/`disconnect` events or `GamepadHandler.gamepads`.

### Methods ###

```javascript
gamepad.addEventListener(event, callback[, index])
```

Adds an event listener to this gamepad. If `index` is supplied, the listener will apply only to events for the button at that index. In the case of `'joystickmove'` events, a two-item array must be passed to `index` to handle two-axis joysticks.

```javascript
gamepad.removeEventListener(event, callback[, index])
```

Removes an event listener from this gamepad. If `index` is supplied, the listener will only be removed from the button at that index. In the case of `'joystickmove'` events, a two-item array must be passed to `index` to handle two-axis joysticks.

#### Events ####

* `event 'buttonpress': callback(index)`

   Fires when a button is pressed and passes the button `index` to the callback.

* `event 'buttonrelease': callback(index)`

   Fires when a button is released and passes the button `index` to the callback.

* `event 'buttonvaluechange': callback(index, value)`

   Fires when a button changes and passes the button `index` and `value` to the callback. This value will typically be `0` or `1` except in the case of an axis button like a trigger.

* `event 'axischange': callback(index, value)`

   Fires when a gamepad axis changes and passes the axis `index` and `value` to the callback. If a deadzone was specified for this axis via `gamepad.setAxisDeadzone`, the deadzone will be subtracted from the value to create a smoother feel.

* `event 'joystickmove': callback(indices, values)`

   Fires when a joystick moves outside of the gamepad's deadzone and passes the axis `indices` and axis `values`. Values have the deadzone subtracted from them to create a smoother feel. To use this event, users must specify in `addEventListener` the pair of axis indices corresponding to the joystick's axes.

```javascript
gamepad.update()
```

Updates this gamepad's values and fires events if necessary.

```javascript
gamepad.getButton(index)
```

Gets the button at the given `index`, in form `{ pressed: true/false, value: x }`.

```javascript
gamepad.getAxis(index)
```

Gets the floating point value of the axis at the given `index`.

```javascript
gamepad.isConnected()
```

Checks if this gamepad is connected.

```javascript
gamepad.getMapping()
```

Gets this gamepad's mapping from the internal gamepad object. This value will only be `'standard'` or `''` at time of writing.

```javascript
gamepad.setAxisDeadzone(index, deadzone)
```

Specifies a deadzone to be used for an axis on this gamepad. Joystick axis deadzones should be set with the `gamepad.joystickDeadzone` property. Must be set to range `[0, 1)`.

```javascript
gamepad.getAxisDeadzone(index)
```

Get the deadzone specified for the axis at `index`, or `undefined` if none specified.


### Properties ###

```javascript
gamepad.joystickDeadzone
```

The joystick deadzone to apply. Defaults to `0.10`. Must be set to range `[0, 1)`.

## `StandardMapping` dictionary ##

A dictionary object containing button/axis `index` values for the `standard` gamepad mapping.

# Limitations #

* The HTML5 Gamepad API may perform differently between browsers. Tested on Ubuntu 18.10 with Chrome (71) and Firefox (64); Windows 10 with Chrome (71), Firefox (64), and Edge (44). The Xbox One controller appears with a standard mapping on each of these configurations *except* Firefox on Ubuntu, where the triggers and D-pad are axes instead of buttons.
* Tested only with an Xbox One controller. Many gamepads share the same standard layout and likely work without additional configuration, but non-standard gamepads (like the XB1 controller on Firefox) will need to be mapped separately.
* Browsers behave differently when detecting gamepads. In my tests on Ubuntu, with Chrome the gamepad is detected automatically, but with Firefox the gamepad sometimes needs to be unplugged and plugged back in to be detected. Sometimes, you must press a button for the gamepad to be recognized for the first time.
* Uses ES6 features, but the HTML5 Gamepad API is new itself so this module should not need to be compiled with Babel or similar.

# TODO #

* Button mappings for common controllers
* Haptic Actuator support. Currently an experimental API internally, so a different implementation may be required per browser engine.
* Add a `gamepads.supported` flag to indicate that the Gamepad API is not supported by a browser.
* Better joystick support. Maybe define a Joystick object which would automatically populated by any recognized/user-defined mapping (e.g. `StandardMapping` for standard gamepads).

# Own a gamepad? #

With so many possible configurations of gamepads, browsers, and operating systems out there, I need help to verify support for various gamepad configurations. Please open issues or pull requests if you find any problems with this module or have any suggestions towards improving support.