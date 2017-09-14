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
暂时没有参数，自动对<scrollbar>的父级的大小进行是否展示滚动条
``` bash
<scrollbar>
  <div></div>
</scrollbar>
```

For detailed explanation on how things work, checkout the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).
