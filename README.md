# v-scrollbar

> Vue-based component, an adaptive scroll bar that displays a scroll bar when the child element's content exceeds the parent scroll bar (基于vue的组件，自适应的滚动条,当子元素的内容超出父级滚动条就会显示滚动条)

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

## Props 

| Props         |Tpye       | Default  |Description                                                                                           |
|---------------|-----------|----------|------------------------------------------------------------------------------------------------------|
| changeTop     | Number    | Null     |change scroll bar position about top (双向绑定顶部的偏移量)        								      |
| changeLeft    | Number    | Null     | change scroll bar position about letf (双向绑定左侧部的偏移量)                                       |
| timeOut       | Number    | 0        | Animation time for subcomponents (如果该组件内有动画会影响这个组件的功能，所有请传入该动画的结束)    |


## Usage
  
``` html
<div>
    <scrollbar @top="top" @bottom="bottom" @left="left" @right="right" :istime="istime" :changeTop.sync="changeTop" :changeLeft.sync="changeLeft">HTML(滚动的内容)</scrollbar>
<div>
```
  
## Events

#### The callback function at the border

| Event         | Output     | Description                                                            |
|---------------|------------|------------------------------------------------------------------------|
| top           | Function   | When the scroll bar is on the topmost side (当滚动条到顶部时的回调)    |
| bottom        | Function   | When the scroll bar is on the bottommost side (当滚动条到底部时的回调) |
| left          | Function   | When the scroll bar is on the leftmost side (当滚动条到最左边时的回调)   |
| right         | Function   | When the scroll bar is on the rightmost side (当滚动条到最右边时的回调)  |
