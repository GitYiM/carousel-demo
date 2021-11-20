<template>
  <div class="swiper">
    <div
      ref="containerRef"
      :style="{
        width: swiperListWidth + 'px',
        transform: 'translateX(' + translateX + 'px)',
        transitionDuration: transitionDuration + 's',
      }"
      class="container"
    >
      <div v-for="(item, index) in imgArr" :key="index">
        <img :src="item.src" alt="" />
      </div>
    </div>
    <div class="dot">
      <span
        v-for="(x, i) in sum"
        :key="'dot' + x"
        :style="{
          background: i === dotIndex ? '#fff' : 'rgba(255, 255, 255, .5)',
        }"
        :class="[i === dotIndex ? 'on' : '']"
      >
      </span>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  defineComponent,
  ref,
  computed,
  onMounted,
} from "@vue/composition-api";
import img1 from "../assets/img1.jpg";
import img2 from "../assets/img2.jpg";
import img3 from "../assets/img3.jpg";
export default defineComponent({
  setup() {
    const curIndex = ref(0);
    const containerRef = ref(null);
    const imgArr = [{ src: img1 }, { src: img2 }, { src: img3 }];
    const sum = imgArr.length;
    const swiperWidth = 520;
    const transitionDuration = ref(0.5);
    const isAutoPlay = false;
    let startX = 0; // touchStart起始坐标
    let offset = 0; // move偏移值
    let timer = null;

    const go = () => {
      curIndex.value++;
      containerRef.value.style.transition = `all ${transitionDuration.value}s ease`;
      if (curIndex.value === sum) {
        setTimeout(() => {
          curIndex.value = 0;
          containerRef.value.style.transition = "none";
        }, 500);
      }
    };

    const back = () => {
      curIndex.value--;
      // containerRef.value.style.transition = `all ${transitionDuration.value}s ease`;
      if (curIndex.value === -1) {
        curIndex.value = sum;
        containerRef.value.style.transition = "none";
        setTimeout(() => {
          curIndex.value = sum - 1;
          containerRef.value.style.transition = `all ${transitionDuration.value}s ease`;
        }, 0);
      }
    };

    const autoPlay = () => {
      timer = setInterval(() => {
        back();
      }, 1000);
    };
    const translateX = computed(() => {
      return -1 * swiperWidth * curIndex.value;
    });
    const swiperListWidth = computed(() => {
      return swiperWidth * (sum + 1);
    });
    const dotIndex = computed(() => {
      if (curIndex.value === sum) {
        return 0;
      }
      return curIndex.value;
    });

    const touchStart = (e: Event) => {
      timer && clearInterval(timer);
      containerRef.value.style.transition = `none`;
      startX = e.targetTouches[0].clientX;
    };

    const touchMove = (e: Event) => {
      offset = startX - e.targetTouches[0].clientX;
      containerRef.value.style.transform = `translateX(${
        translateX.value - offset
      }px)`;
    };

    const touchEnd = () => {
      containerRef.value.style.transition = `all ${transitionDuration.value}s ease`;
      let num = Math.round(offset / swiperWidth);
      let curSum = num + curIndex.value;
      // if (curSum > sum - 1) {
      //   curSum = 0;
      // } else if (curSum < 0) {
      //   curSum = sum - 1;
      // }
      if (curSum === curIndex.value) {
        containerRef.value.style.transform = `translateX(${translateX.value}px)`;
      } else {
        console.log("curSum", curSum);
        curIndex.value = curSum;
        if (curIndex.value === sum) {
          setTimeout(() => {
            curIndex.value = 0;
            containerRef.value.style.transition = "none";
          }, 500);
        }
        if (curIndex.value === -1) {
          curIndex.value = sum;
          containerRef.value.style.transition = "none";
          setTimeout(() => {
            curIndex.value = sum - 1;
            containerRef.value.style.transition = `all ${transitionDuration.value}s ease`;
          }, 0);
        }
      }

      console.log(curIndex);
      offset = 0;
      isAutoPlay && autoPlay();
    };

    onMounted(() => {
      const firstItem = containerRef.value.firstElementChild.cloneNode(true);
      containerRef.value.appendChild(firstItem);
      isAutoPlay && autoPlay();
      containerRef.value.addEventListener("touchstart", touchStart);
      containerRef.value.addEventListener("touchmove", touchMove);
      containerRef.value.addEventListener("touchend", touchEnd);
    });
    return {
      curIndex,
      imgArr,
      containerRef,
      swiperListWidth,
      translateX,
      transitionDuration,
      sum,
      dotIndex,
    };
  },
});
</script>

<style lang="stylus">
.swiper
    width 520px
    height 280px
    margin 0px auto
    position relative
    overflow hidden
    .container
        display flex
        height 100%
        width 100%
        &>div
          height 100%
          width 100%
        img
          height 100%
          width 100%
    .dot
      display flex
      position absolute
      box-sizing border-box
      width 100%
      margin-top -15px
      justify-content flex-end
      padding-right 50px
      span
        width 8px
        height 8px
        background-color rgba(255, 255, 255, .5)
        border-radius 50%
        margin-left 5px
      .on
        width 12px
        border-radius 4px
        transition width 0.3s linear
</style>
