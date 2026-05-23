<template>
  <div v-if="visible" class="footer" :class="{ isWhite }">
    <div class="l-button" @click="refresh(1)">
      <span v-if="!isRefresh1" :class="{ active: currentTab === 1 }">首页</span>
      <img v-if="isRefresh1" src="../assets/img/icon/refresh1.png" alt="" class="refresh" />
    </div>
    <div class="l-button" @click="tab(2)">
      <span :class="{ active: currentTab === 2 }">消息</span>
    </div>
    <div class="l-button" @click="tab(3)">
      <span :class="{ active: currentTab === 3 }">我</span>
    </div>
  </div>
</template>

<script>
import bus, { EVENT_KEY } from '../utils/bus'

export default {
  name: 'BaseFooter',
  props: ['initTab', 'isWhite'],
  data() {
    return {
      isRefresh1: false,
      currentTab: this.initTab,
      visible: true
    }
  },
  created() {
    bus.on('setFooterVisible', (e) => (this.visible = e))
    bus.on(EVENT_KEY.ENTER_FULLSCREEN, () => (this.visible = false))
    bus.on(EVENT_KEY.EXIT_FULLSCREEN, () => (this.visible = true))
  },
  unmounted() {
    bus.off(EVENT_KEY.ENTER_FULLSCREEN)
    bus.off(EVENT_KEY.EXIT_FULLSCREEN)
  },
  methods: {
    $nav(path) {
      this.$router.push(path)
    },
    tab(index) {
      switch (index) {
        case 1:
          this.$nav('/home')
          break
        case 2:
          this.$nav('/message')
          break
        case 3:
          this.$nav('/me')
          break
      }
    },
    refresh(index) {
      if (this.currentTab === index) {
        this['isRefresh' + index] = !this['isRefresh' + index]
        setTimeout(() => {
          this['isRefresh' + index] = !this['isRefresh' + index]
        }, 2000)
      } else {
        this.tab(index)
      }
    }
  }
}
</script>

<style scoped lang="less">
@import '../assets/less/index';

.footer {
  font-size: 14rem;
  position: fixed;
  width: 100%;
  height: var(--footer-height);
  //border-top: 1px solid #7b7878;
  z-index: 2;
  //不用bottom：0是因为，在进行页面切换的时候，vue的transition
  // 会使footer的bottom：0失效，不能准确定位
  top: calc(var(--vh, 1vh) * 100 - var(--footer-height));
  background: var(--footer-color);
  color: var(--jx-text);
  display: flex;
  border-top: 1px solid rgba(225, 230, 238, 0.86);
  box-shadow: 0 -8rem 24rem rgba(18, 27, 44, 0.06);
  backdrop-filter: blur(18rem);

  &.isWhite {
    background: var(--footer-color) !important;
    color: var(--jx-text) !important;
  }

  .l-button {
    width: calc(100% / 3);
    display: flex;
    justify-content: center;
    align-items: center;
    position: relative;
    font-size: 16rem;

    .refresh {
      width: 25%;
      animation: rotate 0.5s linear infinite;
    }

    @keyframes rotate {
      from {
        transform: rotate(0deg);
      }

      to {
        transform: rotate(-360deg);
      }
    }

    span {
      cursor: pointer;
      font-weight: 600;
      opacity: 0.62;
      transition: all 0.2s;

      &.active {
        opacity: 1;
        color: var(--jx-text);
      }
    }
  }
}
</style>
