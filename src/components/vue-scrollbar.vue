<!--
@desc 自定义右下滚动条
会按照div的100%宽高显示，如果内容超出则显示滚动条
e.g
<div><scrollbar></scrollbar></div>
author https://github.com/toMatthew/vue-scrollbar
v2 算法更新换骨
v3 增加了自定义偏移量
v4 增加因子元素有动画音响了他的计算，增加参数timeout，时间结束在执行changesize
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
            },
            istime : true,//优化 因为 频繁 触发 resize 函数，导致页面很卡的 问题
        }
    },
    props:{
        speed: {
            type: Number,
            default: 53
        },
        //上下偏移量 px
        changeTop : {
            type : Number,
            default: null
        },
        //左右偏移量 xp
        changeLeft : {
            type : Number,
            default: null
        },
        timeOut : {//延时调用changeWinSize
            type : Number,
            default: 0
        },
    },
    watch: {
        // 当外面传这个发送变化时就让到这个位置
        changeTop() {  
            if(this.top == this.changeTop) return false;
            let size = this.getSize();
            this.top = this.changeTop;
            this.barTop = (( this.top * 100 ) / size.containerHeight).toFixed(2) * 1;
        },
        changeLeft() { 
            if(this.left == this.changeLeft) return false;
            let size = this.getSize();
            this.left = this.changeLeft;
            this.barLeft = (( this.left * 100 ) / size.containerWidth).toFixed(2) * 1;
        },
    },
    computed:{//计算属性的结果会被缓存，除非依赖的响应式属性变化才会重新计算。

    },
    methods:{
        // 滚动
       scroll(e){
            if(this.isVerticalBtn  || this.isHorizontalBtn) {
                e.preventDefault();
                e.stopPropagation();
            } else {
                return false;
            }          
            let speed = this.speed;
            let shifted = e.shiftKey
            let scrollY = e.deltaY > 0 ? speed*1 : speed * -1;
            // let scrollX = e.deltaX > 0 ? speed : speed * -1;
            // Fix Mozilla Shifted Wheel~
            if(shifted && e.deltaX == 0) scrollX = e.deltaY > 0 ? speed : speed * -1;
            let nextY = this.top*1 + scrollY;
            let nextX = this.left*1 + scrollY;

            // 如果没有垂直的滚动条就滚动横向的
            if(this.isVerticalBtn) {
                this.setVerticalScroll(nextY)
            } else if(this.isHorizontalBtn){
                this.setHorizontalScroll(nextX)
            }
        },
        // 垂直btn点击后的事件
        startmoveV(e) {
            e.preventDefault();//阻止默认事件，取消文字选中
            let _point = this.windowToBox(e.clientX , e.clientY, this.$refs.verticalBtn);//得出来的是相对这垂直btn的点位置
            this.isStartmoveV = true;            
            this.point.x = _point.x;
            this.point.y = _point.y;

            document.addEventListener('mousemove', this.fnMousemoveV, false);
            document.addEventListener('mouseup', this.fnMouseup, false);
        },
        // 点击横向滚动条拖拽滚动
        startmoveH (e) {
            e.preventDefault();//阻止默认事件，取消文字选中
            let _point = this.windowToBox(e.clientX , e.clientY, this.$refs.horizontalBtn);
            this.isStartmoveH = true;
            this.point.x = _point.x;
            this.point.y = _point.y;

            document.addEventListener('mousemove', this.fnMousemoveH, false);
            document.addEventListener('mouseup', this.fnMouseup, false);
        },
        // 垂直移动监听
        fnMousemoveV(e) {
            e.preventDefault();
            if(this.isStartmoveV) {
                this.setVerticalClick(e.clientY - this.point.y);
            }
        },
        // 横向移动监听
        fnMousemoveH(e) {
            e.preventDefault();
            if(this.isStartmoveH) {
                this.setHorizontalClick(e.clientX - this.point.x);
            }
        },
        // 鼠标抬起监听
        fnMouseup(e) {
            e.preventDefault();
            this.isStartmoveH = false;
            this.isStartmoveV = false;
            this.clearMousemove();

        },
        // 清除监听
        clearMousemove() {
            document.removeEventListener('mousemove', this.fnMousemoveV, false);
            document.removeEventListener('mousemove', this.fnMousemoveH, false);
            document.removeEventListener('mouseup', this.fnMouseup, false);
        },
        // 包围盒的信息坐标轴
        windowToBox(x, y, el) {
            let _bbox = el.getBoundingClientRect();
            return {
                x: x - _bbox.left ,
                y: y - _bbox.top
            }
        },
        // 返回盒子尺寸
        getSize(){            
            let container = this.$refs.container;
            let box = this.$refs.box;
            let size = {
                // 滚动内容的高度宽度
                containerHeight: container.clientHeight,
                containerWidth: container.clientWidth,
                // 最外面盒子的高度宽度
                boxHeight: box.clientHeight,
                boxWidth: box.clientWidth,
            }
            
            return size;
        },
        // 点击拖拽设置
        setVerticalClick(val) {
            let size = this.getSize();
            let barTop = ((val - this.boxPoint.minY)  / this.$refs.box.clientHeight ) *100;
            if(barTop <= 0) {
                barTop = 0;
                if(this.barTop == barTop) {
                    return false;
                }
                this.$emit('top');
            }
            if(barTop + this.barHeight >= 100) {
                barTop = 100 - this.barHeight;
                if(this.barTop == barTop) {
                    return false;
                }
                this.$emit('bottom');
            }
            this.barTop = barTop.toFixed(2); // 这里是百分比的需要转换
            this.top = ((barTop / 100) * size.containerHeight).toFixed(2) * 1;
            this.$emit('update:changeTop', this.top);
        },
        // val是偏移量 滚动的 
        setVerticalScroll(val){
            let size = this.getSize();
            let topEnd = size.containerHeight - size.boxHeight;
            if(val >= topEnd){
                val = topEnd;
                if(this.top == val) {// 已经到底部就不用继续执行了
                    return false;
                }
                this.$emit('bottom');
            };
            if(val <= 0) {
                val = 0;
                if(this.top == val) {// 已经到顶部就不用继续执行了
                    return false;
                }
                this.$emit('top');
            }
            this.top = val;
            this.$emit('update:changeTop', this.top);
            // 导航条的top的计算 
            this.barTop = (val / size.containerHeight)*100;
            // (val / size.boxHeight)*100 > (100 - self.barHeight) ? (100 - self.barHeight) : (val / size.boxHeight)*100;
        },
        setHorizontalClick(val) {
            let size = this.getSize();
            let barLeft = ((val - this.boxPoint.minX)  / this.$refs.box.clientWidth ) *100;
            if(barLeft <= 0) {
                barLeft = 0;
                if(this.barLeft == barLeft) {
                    return false;
                }
                this.$emit('left');
            }
            if(barLeft + this.barWidth >= 100) {
                barLeft = 100 - this.barWidth;
                if(this.barLeft == barLeft) {
                    return false;
                }
                this.$emit('right');
            }
            this.barLeft = barLeft.toFixed(2); 
            // 这里是百分比的需要转换
            this.left = ((barLeft / 100) * size.containerWidth).toFixed(2) * 1;
            this.$emit('update:changeLeft', this.left);
        },
        setHorizontalScroll(val){
            let size = this.getSize();
            let leftEnd = size.containerWidth - size.boxWidth;
            if(val >= leftEnd){
                this.$emit('right');
                val = leftEnd;
            };
            if(val <= 0) {
                this.$emit('left');
                val = 0;
            }
            this.left = val;
            this.$emit('update:changeLeft', this.left);
            this.barLeft = (val / size.containerWidth)*100;
            // (val / size.boxWidth)*100 > (100 - self.barWidth) ? (100 - self.barWidth) : (val / size.boxWidth)*100;
        },
        // 改变dom，拉伸窗体或加载内容，dom发生变化
        changeWinSize(){
            if(this.istime) {//优化 因为 频繁 触发 resize 函数，导致页面很卡的 问题
                this.istime = false;

                let size = this.getSize();
                let boxPoint = this.$refs.box.getBoundingClientRect();// container的极坐标

                // 保存极坐标
                this.boxPoint.minX = boxPoint.left;
                this.boxPoint.maxX = boxPoint.right;
                this.boxPoint.minY = boxPoint.top;
                this.boxPoint.maxY = boxPoint.bottom;
                // 计算拖拽条的宽高
                this.barHeight = (size.boxHeight / size.containerHeight) * 100;
                this.barWidth = (size.boxWidth / size.containerWidth) * 100;
                // 是否显示拖拽条
                this.isVerticalBtn = (this.barHeight >= 100 && !!this.barHeight)  ? false : true;
                this.isHorizontalBtn = (this.barWidth >= 100 && !!this.barWidth) ? false : true;
                if(!this.isVerticalBtn) {
                    this.top = 0;
                    this.barTop = 0;
                } 
                if(!this.isHorizontalBtn) {
                    this.left = 0;
                    this.barLeft = 0;
                }
                this.istime = true; 
            }
        }
    },
    mounted () {
        this.$nextTick(()=>{
            this.changeWinSize();
            window.addEventListener('resize', this.changeWinSize);
        });
    },
    updated () {//由于数据更改导致的虚拟 DOM 重新渲染和打补丁，在这之后会调用该钩子
        this.$nextTick(()=>{
            if(this.timeOut != 0) {//防止某些动画影响如el-collapse-transition
                setTimeout(()=>{
                    this.changeWinSize();  
                }, this.timeOut); 
            } else {
                this.changeWinSize();    
            }
        });
    },
    beforeDestroy () {//vue children' of undefined ref 因为还在监听不再这个页面的时候
        window.removeEventListener('resize', this.changeWinSize);
    }
}
</script>

<style scoped>
.scrollbar_box{overflow: hidden; position: relative; height: 100%; width: 100%;}
.scrollbar_container{overflow: visible; min-width: 100%; min-height: 100%;}
.scrollbar_verticalBtn{position: absolute; top: 0; right: 1px; width: 6px; border-radius: 6px; background-color: rgba(153, 153, 153, .3); cursor: pointer;}
.scrollbar_horizontalBtn{position: absolute; bottom: 2px; left: 0; height: 6px; border-radius: 6px; background-color: rgba(153, 153, 153, .5); cursor: pointer;}
.scrollbar_box:hover .scrollbar_verticalBtn,.scrollbar_box:hover .scrollbar_horizontalBtn{background-color: rgb(153, 153, 153)}
</style>