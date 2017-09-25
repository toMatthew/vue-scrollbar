# v-scrollbar

> A Vue.js project

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

```
## Install
``` bash
npm install v-scrollbar --save-dev
```

``` bash
import scrollbar from 'v-scrollbar';
Vue.component('my-component', {
    components: {
        scrollbar
    }
});
```

## Usage
  
``` html
<scrollbar>
  <div>HTML</div>
</scrollbar>
```
  
## Events

#### The callback function at the border

| Event         | Output     | Description                                    |
|---------------|------------|------------------------------------------------|
| top           | Function   | When the scroll bar is on the topmost side     |
| bottom        | Function   | When the scroll bar is on the bottommost side  |
| left          | Function   | When the scroll bar is on the leftmost side    |
| right         | Function   | When the scroll bar is on the rightmost side   |

``` html
<datepicker @top="top" @bottom="bottom" @left="left" @right="right"></datepicker>
```