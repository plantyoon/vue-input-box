# vue-input-box

Input Box Component inspired by Gmail email address input

## Install
```
npm install vue-input-box
yarn add vue-input-box
```

## Use
```html
  <input-box v-model="['item-1', 'item-2', 'item-3'].map(item => ({ value: item }))" filter="^[\w-]*$" placeholder="Add Item" />
```

## Props
|Name|Type|default|Description|
|----|:----:|:----:|:----:|
|value|Array|[]|Default box settings. Must be an array of type {value: String}|
|filter|String|''|Filter for input values.|
|placeholder|String|''|Placeholder exposed when input is blank.|
|alert|String|''|Notification message when you enter a value that does not match filter.|