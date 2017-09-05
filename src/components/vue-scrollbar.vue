<!--
@desc 自定义右下滚动条
会按照div的100%宽高显示，如果内容超出则显示滚动条
e.g
<div><scrollbar></scrollbar></div>
author https://github.com/toMatthew/vue-scrollbar
v2 算法更新换骨
-->
<template>
<div class="scrollbar_box" @wheel="scroll" ref="box">
    <div class="scrollbar_container" :style="{ transform : 'translate(' + left*-1 +'px,' + top*-1 +'px)' }" ref="container">
        <slot></slot>      
    </div>
    <div class="scrollbar_verticalBtn" :style="{height: barHeight + '%', top : barTop + '%'}" v-show="isVerticalBtn" @mousedown="startmoveV" ref="verticalBtn"><!--竖直滚动条-->
    </div>
    <div class="scrollbar_horizontalBtn" :style="{width: barWidth + '%', left : barLeft + '%'}" v-show="isHorizontalBtn" @mousedown="startmoveH" ref="horizontalBtn"><!--横向滚动条-->
    </div>
</div>  
</template>

<script>
export default {
    data: function () {
        return {
            top : 0, 
            left : 0, 
            barHeight : 0, //竖直滚动条高度
            barWidth : 0, //宽度横向滚动条
            barTop : 0,//竖直滚动条上下偏移量
            barLeft : 0,//横向滚动条左右偏移量
            isVerticalBtn : false, //竖直滚动条显示
            isHorizontalBtn : false, //横向滚动条显示
            isStartmoveV:false,//点击拖拽垂直
            isStartmoveH:false,//点击拖拽横向
            point: {//相对的x,y坐标
                x:0,
                y:0,
            },
            boxPoint : {// 4个角容器的极坐标
                minX:0,
                maxX:0,
                minY:0,
                maxY:0,
            }
        }
    },
    props:{
        // containerStyle : Object,
        speed: {
            type: Number,
            default: 53
        },
        //上下偏移量 %
        topChange : {
            type : Number,
            default: 0
        },
        //左右偏移量 %
        leftChange : {
            type : Number,
            default: 0
        },
        styles : {
            type : Number,
            default : 2,
        },
        /*
            设置2种模式
            1、内容改变时将top设为0
            2、内容改变通过计算如果e.g
                    原来高100，top为50%--> 变为搞200 --> top就应该是25%
        */
    },
    computed:{//计算属性的结果会被缓存，除非依赖的响应式属性变化才会重新计算。

    },
    watch:{

    },
    methods:{
        // 滚动
       scroll:function(e){
            var self = this;
            if(self.isVerticalBtn  || self.isHorizontalBtn) {
                e.preventDefault();
                e.stopPropagation();
            } else {
                return false;
            }          
            var speed = self.speed;
            var shifted = e.shiftKey
            var scrollY = e.deltaY > 0 ? speed*1 : speed * -1;
            // var scrollX = e.deltaX > 0 ? speed : speed * -1;
            // Fix Mozilla Shifted Wheel~
            if(shifted && e.deltaX == 0) scrollX = e.deltaY > 0 ? speed : speed * -1;
            var nextY = self.top*1 + scrollY;
            var nextX = self.left*1 + scrollY;

            // 如果没有垂直的滚动条就滚动横向的
            if(self.isVerticalBtn) {
                self.setVerticalScroll(nextY)
            } else if(self.isHorizontalBtn){
                self.setHorizontalScroll(nextX)
            }
            
        },
        // 垂直btn点击后的事件
        startmoveV: function(e) {
            e.preventDefault();//阻止默认事件，取消文字选中
            var self = this;
            self.isStartmoveV = true;
            var point = self.windowToBox(e.clientX , e.clientY, self.$refs.verticalBtn);//得出来的是相对这垂直btn的点位置
            self.point.x = point.x;
            self.point.y = point.y;
            if(!e){
                e = window.event;
                //防止IE文字选中
                self.$refs.box.onselectstart = function(){
                    return false;
                }  
            }

            document.onmousemove = function (e) {
                e.preventDefault();
                if(self.isStartmoveV) {
                    self.setVerticalClick(e.clientY - self.point.y);
                }
            }

            document.onmouseup = function (e) {
                e.preventDefault();
                self.isStartmoveV = false;
                clear();
            }

            var clear = function () {
                document.onmouseup = null;
                document.onmousemove = null;
            }
        },
        // 点击横向滚动条拖拽滚动
        startmoveH : function (e) {
            e.preventDefault();//阻止默认事件，取消文字选中
            var self = this;
            self.isStartmoveH = true;
            var point = this.windowToBox(e.clientX , e.clientY, self.$refs.horizontalBtn);
            self.point.x = point.x;
            self.point.y = point.y;
            if(!e){
                e = window.event;
                //防止IE文字选中
                self.$refs.box.onselectstart = function(){
                    return false;
                }  
            }

            document.onmousemove = function (e) {
                e.preventDefault();
                if(self.isStartmoveH) {
                    self.setHorizontalClick(e.clientX - self.point.x);
                }
            }

            document.onmouseup = function (e) {
                e.preventDefault();
                self.isStartmoveH = false;
                clear();
            }

            var clear = function () {
                document.onmouseup = null;
                document.onmousemove = null;
            }
        },
        windowToBox : function(x, y, el) {
            var bbox = el.getBoundingClientRect();///canvas的包围盒的信息
            return {x:x-bbox.left , y:y-bbox.top}
        },
        getSize: function(){            
            var container = this.$refs.container;
            var box = this.$refs.box;

            var size = {
                //滚动内容的高度宽度
                containerHeight: container.children[0].clientHeight,
                containerWidth: container.children[0].clientWidth,
                //最外面盒子的高度宽度
                boxHeight: box.clientHeight,
                boxWidth: box.clientWidth,
            }
            
            return size;
        },
        setVerticalClick(val) {//点击拖拽设置
            var self = this;
            var size = self.getSize();
            var barTop = ((val - self.boxPoint.minY)  / this.$refs.box.clientHeight ) *100;
            if(barTop < 0) {
                barTop = 0;
                if(self.barTop == barTop) {
                    return false;
                }
                self.$emit('top');
            }
            if(barTop + self.barHeight > 100) {
                barTop = 100 - self.barHeight;
                if(self.barTop == barTop) {
                    return false;
                }
                self.$emit('bottom');
            }
            self.barTop = barTop.toFixed(2); //这里是百分比的需要转换
            self.top = ((barTop / 100) * size.containerHeight).toFixed(2) * 1;
        },
        setVerticalScroll(val){//val是偏移量 滚动的 
            var self = this;
            var size = self.getSize();
            let topEnd = size.containerHeight - size.boxHeight;
            if(val > topEnd){
                val = topEnd;
                if(self.top == val) {//已经到底部就不用继续执行了
                    return false;
                }
                self.$emit('bottom');
            };
            if(val < 0) {
                val = 0;
                if(self.top == val) {//已经到顶部就不用继续执行了
                    return false;
                }
                self.$emit('top');
            }
            self.top = val;
            // 导航条的top的计算 
            self.barTop = (val / size.containerHeight)*100;//(val / size.boxHeight)*100 > (100 - self.barHeight) ? (100 - self.barHeight) : (val / size.boxHeight)*100;
        },
        setHorizontalClick(val) {
            var self = this;
            var size = self.getSize();
            var barLeft = ((val - self.boxPoint.minX)  / this.$refs.box.clientWidth ) *100;
            if(barLeft < 0) {
                barLeft = 0;
                if(self.barLeft == barLeft) {
                    return false;
                }
                self.$emit('left');
            }
            if(barLeft + self.barWidth > 100) {
                barLeft = 100 - self.barWidth;
                if(self.barLeft == barLeft) {
                    return false;
                }
                self.$emit('right');
            }
            self.barLeft = barLeft.toFixed(2); //这里是百分比的需要转换
            self.left = ((barLeft / 100) * size.containerWidth).toFixed(2) * 1;
        },
        setHorizontalScroll(val){
            var self = this;
            var size = self.getSize();
            let leftEnd = size.containerWidth - size.boxWidth;
            if(val > leftEnd){
                self.$emit('right');
                val = leftEnd;
            };
            if(val < 0) {
                self.$emit('left');
                val = 0;
            }
            self.left = val;
            self.barLeft = (val / size.containerWidth)*100;//(val / size.boxWidth)*100 > (100 - self.barWidth) ? (100 - self.barWidth) : (val / size.boxWidth)*100;
        },
        changeWinSize(){
            var self = this;
            var size = self.getSize();
            var boxPoint = self.$refs.box.getBoundingClientRect();//container的极坐标
            // 保存极坐标
            self.boxPoint.minX = boxPoint.left;
            self.boxPoint.maxX = boxPoint.right;
            self.boxPoint.minY = boxPoint.top;
            self.boxPoint.maxY = boxPoint.bottom;
            // 计算拖拽条的宽高
            self.barHeight = (size.boxHeight / size.containerHeight) * 100;
            self.barWidth = (size.boxWidth / size.containerWidth) * 100;
            // 是否显示拖拽条
            self.isVerticalBtn = (self.barHeight >= 100 && !!self.barHeight)  ? false : true;
            self.isHorizontalBtn = (self.barWidth >= 100 && !!self.barWidth) ? false : true;
            if(!self.isVerticalBtn) {
                self.top = 0;
                self.barTop = 0;
            } 
            if(!self.isHorizontalBtn) {
                self.left = 0;
                self.barLeft = 0;
            }
        }
    },
    mounted () {
        this.changeWinSize();
        window.addEventListener('resize', this.changeWinSize);
    },
    updated () {//由于数据更改导致的虚拟 DOM 重新渲染和打补丁，在这之后会调用该钩子
        this.$nextTick(() => {
        // setTimeout(()=>{//防止某些动画影响如el-collapse-transition
            this.changeWinSize();      
        // },150);  
        });  
    },
    beforeDestroy () {//vue children' of undefined ref 因为还在监听不再这个页面的时候
      window.removeEventListener('resize', this.changeWinSize);
    }
}
</script>

<style scoped>
.scrollbar_box{overflow: hidden; position: relative; height: 100%; width: 100%;}
.scrollbar_container{overflow: visible;}
.scrollbar_verticalBtn{position: absolute; top: 0; right: 0; width: 6px; border-radius: 6px; background-color: rgba(153, 153, 153, .3); cursor: pointer;}
.scrollbar_horizontalBtn{position: absolute; bottom: 0; left: 0; height: 6px; border-radius: 6px; background-color: rgba(153, 153, 153, .5); cursor: pointer;}
.scrollbar_box:hover .scrollbar_verticalBtn,.scrollbar_box:hover .scrollbar_horizontalBtn{background-color: rgb(153, 153, 153)}
</style>

