<template>
  <div>
    <div
      class="__Carousel"
    >
      <section
        v-for="(item,index) in data"
        :class="{
          slide:true,
          left:currentSectionPos>index,
          active:currentSectionPos===index, 
          right:currentSectionPos<index,
          }"
        :style="{
            zIndex:zIndex(index)
          }"
        :key="index"
        @mouseover="doPause"
        @mouseout="doContinue"
        @mouseup="handleClick($event,value[index],index)"
        @click="focusOn(index)"
      >
        <figure>
          <img
            :src="item.pic"
            :style="{
              width:(item.width&&item.width!==0)?`${item.width}px` :'100%',
              height:(item.height&&item.height!==0)?`${item.height}px` :null
            }"
            alt=""
          >
        </figure>
      </section>
      <div
        @click="toLeft"
        class="slide-arrow left"
      ></div>
      <div
        @click="toRight"
        class="slide-arrow right"
      ></div>
      <div v-if="indicators" class="slide-indicators">
        <i
          @click="focusOn(index)"
          v-for="(i,index) in value"
          :key="index"
          :class="{'slide-indicator':true,active:index===currentSectionPos}"
          @mouseover="doPause"
          @mouseout="doContinue"
        />
      </div>
    </div>
  </div>
</template>

<script>
import { onMounted, ref, nextTick, reactive, watch } from 'vue'
export default {
  name: 'MPCarousel',
  props: {
    value: {
      type: Array,
      default: () => []
    },
    indicators:{
      type:Boolean,
      default:()=>false
    }
  },
  emits: ['click'],
  setup (props, context) {
    var data = reactive([])
    watch(()=>props.value,(val)=>{
      var sortedArr = val.sort((pre,next)=>pre.order-next.order)
      sortedArr.map(i=>data.push(i))  
    },{ 
      immediate:true
    }) 
    var currentSectionPos = ref(0);
    var pause = ref(false);
    var playTimer = ref(null);
    var intervalTime = ref(4000);
    var focusOn = (index) => {
      currentSectionPos.value = index
    };
    var convert = (number) => {
      if (typeof number === 'number') {
        return `${number}px`
      }
      if (typeof number === 'string') {
        if (number.indexOf('%') > 0) {
          return `${number.split('%')[0]}%`
        } else {
          return number
        }
      }
    }
    var doPause = () => {
      context.emit('pause',currentSectionPos.value)
      pause.value = true
    }
    var doContinue = () => {
      pause.value = false
    }
    var play = () => {
      clearTimeout(playTimer.value)
      playTimer.value = setTimeout(function () {
        if (!pause.value) {
          var newPos = (currentSectionPos.value + 1) % data.length
          focusOn(newPos)
        }
        play()
      }, intervalTime.value)
    }
    var toRight = () => {
      var newPos = (currentSectionPos.value + 1) % data.length
      context.emit('next',newPos)
      focusOn(newPos)
    }
    var toLeft = () => {
      var newPos = (currentSectionPos.value + data.length - 1) % data.length
      context.emit('previous',newPos)
      focusOn(newPos)
    }
    var zIndex = (index) => {
      var result = 0
      if (currentSectionPos.value > index) {
        result = index
      } else if (currentSectionPos.value < index) {
        result = data.length - (index - currentSectionPos.value)
      } else {
        result = data.length
      }
      return result
    }
    var handleClick = ($event, item, index) => {
      context.emit('click', $event, item, index)
    }
    onMounted(() => {
      nextTick(() => {
        play()
      })
    })
    return {
      currentSectionPos,
      focusOn,
      toLeft,
      toRight,
      doPause,
      doContinue,
      zIndex,
      handleClick,
      convert,
      data
    }
  }
}
</script>

<style lang="scss" scoped>
.__Carousel {
  min-height: 300px;
  position: relative;
  display: flex;
  align-items: center;
  flex-direction: row;
  cursor: pointer;
  overflow: hidden;
  .slide {
    flex: 1;
    border-radius: 5px;
    transition: all 0.5s;
    background-repeat: no-repeat;
    background-size: contain;
    background-position: center;
    filter: blur(2px);
    /* 强制开启gpu加速 */
    transform: translateZ(0);
    user-select: none;
    &.active {
      pointer-events: none;
      filter: none;
      @media (max-width: 700px) {
        flex: 5;
      }

      @media (min-width: 700px) and (max-width: 1080px) {
        flex: 4;
      }

      @media (min-width: 1080px) {
        flex: 2;
      }
    }
    &.left {
      transform: rotate3d(3, 10, 1, 45deg);
      margin-right: -18%;
    }
    &.right {
      transform: rotate3d(3, -10, -1, 45deg);
      margin-left: -18%;
    }

    & > figure {
      margin: 0;
      padding: 0;
      border: none;
      display: flex;
      justify-content: center;
      & > * {
        background-color: #fff;
        box-shadow: 0px 0px 30px #838383;
        pointer-events: all;
      }
    }
  }
  .slide-arrow {
    position: absolute;
    z-index: 999;
    background-color: rgba(0, 0, 0, 0.2);
    color: #fff;
    line-height: 45px;
    font-size: 50px;
    height: 50px;
    width: 50px;
    border-radius: 25px;
    &:hover {
      background-color: rgba(0, 0, 0, 0.5);
    }
    &.left {
      display: flex;
      align-items: baseline;
      justify-content: flex-end;
      padding-right: 10px;
      left: -30px;
      &::before {
        content: "‹";
      }
    }
    &.right {
      display: flex;
      align-items: baseline;
      justify-content: flex-start;
      padding-left: 10px;
      right: -30px;
      &::before {
        content: "›";
      }
    }
  }
  .slide-indicators {
    display: flex;
    justify-content: center;
    align-items: center;
    position: absolute;
    bottom: 10px;
    width: 100%;
    .slide-indicator {
      width: 5px;
      height: 5px;
      overflow: hidden;
      background: #fff;
      box-shadow: 0 0 10px #333;
      margin: 5px;
      z-index: 9999;
      border-radius: 50%;
      &.active {
        width: 8px;
        height: 8px;
      }
    }
  }
}
</style>