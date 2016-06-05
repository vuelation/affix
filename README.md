# Vuelation Affix

A Vue.js component for affixing an element in place while scrolling.

## Installation

```sh
$ npm install vuelation-affix --save
```

## Usage

Register the Vue component:

```js
Vue.component('affix', require('vuelation-affix'));
```

Use the component in your HTML:

```html
<div class="container">
  <affix>
    <div class="affixed-element">
      This element will be affixed in place when the window is
      scrolled to the top of the container.

      The element will be affixed to the bottom of the container
      when the window is scrolled past the bottom of the container.
    </div>
  </affix>
</div>
```
