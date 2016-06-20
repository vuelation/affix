# Vuelation Affix

A Vue.js component for affixing an element in place while scrolling.

## Installation

Install via [npm](https://www.npmjs.com):

```sh
$ npm install vuelation-affix --save
```

Add [webpack](http://webpack.github.io) config:

```js
{
  module: {
    loaders: [{
      test:    /\.vue$/,
      loaders: ['vue']
    }]
  }
}
```

## Usage

Register the Vue component:

```js
Vue.component('affix', require('vuelation-affix'));
```

Use the component:

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
