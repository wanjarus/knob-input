# Knob Input

> ⚠️ **Note:** This library is in **alpha**. The names and functionality of all classes, properties, and methods are subject to change without notice until the full v1.0 release. See [issues](https://github.com/jhnsnc/knob-input/issues/) for discussion of changes.

`KnobInput` is a vanilla JS class for creating highly customizable, accessible, and usable knob/dial/slider inputs. Right now it is just the flexible base class, but the finished library will also include complete, fully styled wrapper components ready to drop into any project.

DEMO COMING SOON. Until then you can look at [this CodePen](https://codepen.io/jhnsnc/pen/KXYayG) to get a general idea of what the input can do.

These components can be styled to fit in perfectly in any app, and allow users to set precise values through many input modalities. Users can touch-and-drag, click-and-drag, scroll their mouse wheel, double click, or use keyboard input. After instantiation you can use the component just like you would any normal input.

This library is in active development. Please report any issues you discover [on Github](https://github.com/jhnsnc/knob-input/issues).

## Planned Features

- ✔ Customizable, bare-bones base component
- ❌ All-in-one, visually-complete wrapper components
- Support for manipulation via many input modes
  - ✔ mouse drag
  - ✔ touch drag
  - ✔ mouse wheel
  - ✔ mouse double-click (reset)
  - ✔ keyboard input
- Flexible deployment options
  - ✔ Common JS
  - ✔ UMD
  - ❌ ES modules *(see `src/` folder for now for uncompressed ES modules)*
  - ✔ `window` global
  - ❌ React bindings (likely a separate package when implemented)
  - ✔ CSS
  - ❌ Sass
- ❌ Detailed documentation and usage demos

## Basic Usage

1. Include the base component styles from `css/knob-input.css` and the base component script from `scripts/knob-input.js`.

> If you are using CommonJS or UMD modules for your front-end, you can instead use the scripts in `common/` and `umd/` respectively.

2. Create a container element with class `knob-input` and a visual element inside the container with class `knob-input__visual`.

```html
<div class="knob-input">
  <div class="knob-input__visual"></div>
</div>
```

> The container may be any size, and the visual element should be styled how you want the input to look, filling all the available space.

3. Initialize the component in your JS:

```js
var myKnobContainer = document.querySelector('.knob-input');
var myKnobInput = new KnobInput(myKnobContainer);
```

4. Access the value when needed, as you would with a normal input element:

```js
// retrieve value
var currentValue = myKnobInput;

// set value
myKnobInput.value = 0.5;

// watch for changes
myKnobInput.addEventListener('change', function(evt) {
  console.log(evt.target.value);
});
```

## Input Configuration

You can specify a `min` and `max` value in the setup function, as well as a `step` size. These behave the same way as with the standard [range input](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/range) element.

```js
var myKnobInput = new KnobInput(myKnobContainer, {
  min: -100,
  max: 100,
  step: 5
});
```

You can also set an `initial` value and determine how precise drag and mouse wheel actions are with the `dragResistance` and `wheelResistance` options.

```js
var myKnobInput = new KnobInput(myKnobContainer, {
  initial: 0.4, // default = half-way between `min` and `max`
  dragResistance: 500, // default = 300
  wheelResistance: 3000, // default = 4000
});
```

## Event Passthrough

You can add/remove events on the `KnobInput` instance just like you would for a normal `<input>` element.

```js
myKnobInput.addEventListener('change', function(evt) {
  console.log('I changed value:', evt.target.value);
});

myKnobInput.addEventListener('focus', function(evt) {
  console.log('I now have keyboard focus');
});

myKnobInput.addEventListener('blur', function(evt) {
  console.log('I lost keyboard focus');
});
```

## Customizing Visual Appearance

### Drag and Selected States

The document `body` will receive class `knob-input__drag-active` whenever the user is dragging a knob input. The default behavior in this case is to change the cursor to the closed "grabbing" cursor. The input container itself will also receive the class `drag-active` for the duration of the interaction.

The input container will also receive the class `focus-active` whenever the input has focus. This allows the visual element to receive CSS changes based on the focus state even though the hidden input element is actually the element holding the browser's focus (this is done for accessibility reasons).

### Visual Context

You can set a custom `updateVisuals` callback to update the appearance of the input's visual element. Any time the input changes value, this callback will get called with two parameters: the new input value in normalized form (`0` ≤ `n` ≤ `1`) and the actual new value.

```js
var myKnobInput = new KnobInput(myKnobContainer, {
  updateVisuals: function(norm) {
    // rotate the element between 0 and 360 degrees, based on current value
    this.element.style[this.transformProperty] = 'rotate(' + (360*norm) + 'deg)';
    // fade the background color between red and green, based on current value
    this.element.style.backgroundColor = 'rgb(' + Math.floor(256*(1-norm)) + ',' + Math.floor(256*norm) + ',0)';
  }
});
```

Inside the `updateVisuals` callback, the visual element will always be available as `this.element`. This is because the `updateVisuals` callback is bound to an internal visual context. There is also `this.transformProperty` available for convenience, representing the appropriate CSS transform property name for the active web browser.

You can extend the visual context by specifying a `visualContext` function. This gets run once on creation and can be useful for caching DOM elements and calculated values, so that the `updateVisuals` can be more efficient and avoid unnecessary DOM operations, calculations, etc.

```js
var myKnobInput = new KnobInput(myKnobContainer, {
  visualContext: function() {
    // cache DOM elemetn
    this.textDisplay = this.element.querySelector('div.current-value-indicator');
    // purple
    this.minColor = {
      r: 156,
      g: 39,
      b: 175
    };
    // orange
    this.maxColor = {
      r: 255,
      g: 152,
      b: 0
    };
  },
  updateVisuals: function(norm, val) {
    // set the text to the new value
    this.textDisplay.innerText = val;
    // change the text color based on the current value
    var r = this.minColor.r*(1-norm) + this.maxColor.r*norm;
    var g = this.minColor.g*(1-norm) + this.maxColor.g*norm;
    var b = this.minColor.b*(1-norm) + this.maxColor.b*norm;
    this.element.style.color = 'rgb(' + r + ',' + g + ',' + b + ')';
  }
});
```

## All Options

In addition to the `containerElement` parameter of the `KnobInput` constructor, there is an `options` object for you to specify any of the following settings.

| `options` property   | default value | description |
|----------------------|---------------|-------------|
| `min`                | `0` | The minimum input value. Same as with the standard [range input](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/range) element. |
| `max`                | `1` | The maximum input value. Same as with the standard [range input](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/range) element. |
| `step`               | `'any'` | The step amount for value changes. Same as with the standard [range input](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/range) element. |
| `initial`            | average of `min` and `max` | The initial value of the input. Also used when the user double-clicks to reset the input value. |
| `dragResistance`     | `300` | The amount of resistance to value change on mouse/touch drag events. Higher value means more precision, and the user will have to drag farther to change the input's value. |
| `wheelResistance`    | `4000` | The amount of resistance to value change on mouse wheel scroll. Higher value means more precision, and the mouse wheel will be less effective at changing the input's value. |
| `visualContext`      | callback that sets `minRotation` to `0` and `maxRotation` to `360` | Callback that allows for customization of the visual context by setting properties via `this`. Note that `this.element` and `this.transformProperty` will already have values. Useful for caching DOM references, calculations, etc for use in the `updateVisuals` callback. |
| `updateVisuals`      | callback that updates visual element rotation based on `minRotation`/`maxRotation` | Custom callback for updating the input visuals based on changes to the input value. Has access to the visual context via `this` (e.g. `this.element`). |
| `visualElementClass` | `'control-knob__visual'` | **deprecated** This option will likely be removed or changed. It is not recommended to use this. |
