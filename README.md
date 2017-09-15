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
| top           |            | When the scroll bar is on the topmost side     |
| bottom        |            | When the scroll bar is on the bottommost side  |
| left          |            | When the scroll bar is on the leftmost side    |
| right         |            | When the scroll bar is on the rightmost side   |

``` html
<datepicker @top="top" @bottom="bottom" @left="left" @right="right"></datepicker>
```

For detailed explanation on how things work, checkout the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).
