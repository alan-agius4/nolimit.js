# nolimit.js

Javascript game loader and API for operators to load and  communicate with Nolimit games.

## Introduction

**nolimit.js** provides a powerful yet simple way to load games, while still putting most of the control in the hands of the operator.

The operator makes all the design and layout, and will only need to provide a target element for the loader to use.

## Load and initialize

**nolimit.js** is available with sourcemap and some logging as `nolimit-VERSION.js` and as minified as `nolimit-VERSION.min.js` at *\[URL to be decided]*.

**nolimit.js** files are packaged as [UMD](https://github.com/umdjs/umd), meaning it can be loaded using [CommonJS](http://wiki.commonjs.org/wiki/CommonJS) such as [Browserify](http://browserify.org/), [AMD](https://github.com/amdjs/amdjs-api/wiki/AMD) such as [RequireJS](http://requirejs.org/) or just standalone with a regular `<script>` tag.

When loaded standalone, it will add a global variable `window.nolimit` which is used to load the games. See examples below.

### Using CommonJS

```javascript
var nolimit = require('nolimit');
nolimit.init({
    operator: 'SMOOTHOPERATOR'
});
```

### Using AMD

```javascript
define(['nolimit'], function(nolimit) {
    nolimit.init({
        operator: 'SMOOTHOPERATOR'
    });
});
```

### As a global variable

```html
<script src="nolimit-1.0.0.js"></script>
<script>
nolimit.init({
    operator: 'SMOOTHOPERATOR'
});
</script>
```

## Loading a game

### As part of the page

Create an element, usually a `<div>`, that will be *replaced* by the game's `<iframe>`. The iframe will retain styling information such as `width` and `height`, `display` and so on.

```javascript
var api = nolimit.load({
    target: document.querySelector('#game'),
    game: gameName
});

api.on('exit', closeGame);
```

### Replace window

Common use case on mobile, where the game should fill the whole screen and you still want the back button to work. 

Make a separate page which contains the game loading script and navigate to that. `nolimit.load()` will default to replacing the whole current window.

```javascript
var api = nolimit.load({
    game: gameName
});

api.on('exit', function() {
    // or go to specific lobby page, or any other special handling
    history.back();
});
```

It's of course also possible to simply make a full screen `<div>` and use that as target if that fits your flow better. Note though, that you need to make the game element responsive or resize manually in that case.

## Options

It's not strictly required to call `init()`, it's possible to call `load()` directly with all the options instead. `init()` just provides a way to setup some common defaults and the options will actually be merged before each game load.
 
The only required options are the **operator** code for init, the **game** code for game loading - and the HTMLElement** or Window to replace.

### Real money or play money

For playing with real money, a one time **token** is required, that is generated by the operator. The token will be verified via an API call to the operator's server. If the token is missing, fun mode is automatically enabled. If the token is present, but fails validation, there will be an error inside the game.

### init(options)

* `@param {String}  options.operator the operator code for the operator`
* `@param {String}  [options.language="en"] the language to use for the game`
* `@param {String}  [options.device=desktop] type of device: 'desktop' or 'mobile'`
* `@param {String}  [options.environment=partner] which environment to use; usually 'partner' or 'production'`
* `@param {String}  [options.currency=EUR] currency to use, if not provided by server`

```javascript
    nolimit.init({
        operator: operator,
        language: 'sv',
        device: 'mobile',
        environment: 'production',
        currency: 'SEK'
    });
```

### load(options)

* `@param {String}              options.game case sensitive game code, for example 'CreepyCarnival' or 'SpaceArcade'`
* `@param {HTMLElement|Window}  [options.target=window] the HTMLElement or Window to load the game in`
* `@param {String}              [options.token=undefined] the token to use for real money play`
* `@param {Boolean}             [options.mute=false] start the game without sound`
* `@param {Object}              [options.events={}] events from within the game`

```javascript
    nolimit.load({
        game: gameName,
        target: gameElement,
        token: realMoneyToken,
        mute: true,        
        events: {}
    });
```

## Communicating with the game

### Events

The operator can add callbacks for certain events, such as the player trying to close the game or do a deposit from within the game.

```javascript
    var api = nolimit.load({
        game: gameName
    });

    api.on('ready', function onLoad() {
        // ...
    });

    api.on('exit', function goToLobby() {
        // ...
    });

    api.on('deposit', function openDeposit() {
        // ...
    });

    api.on('support', function openHelpChat() {
        // ...
    });
```

* ready - fired when the game is loaded and ready, and the API can be used.
* exit - (mobile only) fired when the player presses the exit button in the game.
* balance - fired when it's safe to update the total balance, as far as this game is concerned. Use only for timing.
* deposit - fired when the player presses the Deposit button in the game.
* support - fired when the player presses the Support button in the game.

## More Examples

### Multiple games

```html
<div class="multigame">
    <div id="game1">
    <div id="game2">
</div>
```

```javascript
nolimit.init({
    operator: 'SMOOTHOPERATOR',
    language: 'sv',
    environment: 'production'
});

var gameElement1 = document.getElementById('game1');
nolimit.load({
    target: gameElement1,
    game: 'KitchenDrama'
});

var gameElement2 = document.getElementById('game2');
nolimit.load({
    target: gameElement2,
    game: 'CreepyCarnival'
});

```

## How do I...?

### Style the game iframe

For instance, set the initial background color, width/height etc.

Style the original element (probably a `<div>`). Note that if you do this with CSS, the tag will be `iframe`, so this selector will not work:

```css
div.game {...}
```

but any of these will:

```css
div.game, iframe.game {...}

.game {...}
```
