<script setup>
import { ref, computed } from 'vue'

import {
  leftSwitchArrow,
  rightSwitchArrow,
  yuRoleInfo,
  roleSidebar,
  bottomRoleBac,
  outerBackground,
  mobileActiveBac,
} from './roleMes'

// currentIndex 控制场景索引大类区分
const currentIndex = ref(0)
const currentRoleInfeo = computed(() => {
  return yuRoleInfo[currentIndex.value]
})
// 最外层两张根据场景不同的照片切换
const outerTwoBackground = computed(() => {
  return outerBackground[currentIndex.value]
})
// 侧边栏点击切换
function changeRole(index) {
  moveDistanceMobile.value = 0
  recordLastMove.value = 0
  currentIndex.value = index
  $index.value = 0
}
//存放角色信息
const yuRoleMes = computed(() => {
  let arr = []
  currentRoleInfeo.value.roleAvatar.forEach((item, index) => {
    arr.push({
      roleAvatar: item,
      roleName: currentRoleInfeo.value.roleName[index],
      roleBackground: isPC.value
        ? currentRoleInfeo.value.roleBackground[index]
        : currentRoleInfeo.value.roleBackgroundMobile[index],
      roleIntroduce: currentRoleInfeo.value.roleIntroduce[index],
    })
  })
  return arr
})

// 背景中文介绍
const roleIntroduce = computed(() => {
  return yuRoleMes.value[$index.value].roleIntroduce
})
// 背景角色切换
const $index = ref(0) //控制哪个背景展示
const roleBackgroundUrl = computed(() => {
  return yuRoleMes.value[$index.value].roleBackground
})
// 背景名字切换
const roleNameSwitch = computed(() => {
  return yuRoleMes.value[$index.value].roleName
})

//#region 底部点击操作
// 每次点击移动距离
const moveDistance = computed(() => {
  // 前三张往右移动,固定位移
  if (isPC.value) {
    const firstThree = $index.value < 3
    if (firstThree) {
      return 0
    }
    // 第一张往左移动 跳到最后一页
    // 后四张往右移动时，如果是后四张就固定位移，不是就移动位移即可
    const lastThree = $index.value > yuRoleMes.value.length - 4 //后三张往左移动
    if (!lastThree) {
      return ($index.value - 2) * -144
    }
    // 固定位移后四张
    return (yuRoleMes.value.length - 6) * -144
  }
})
// 判断是不是前三张||后三张
const moveDistancePC = computed(() => {
  return moveDistance.value + 'px'
})
// 上一页
function lastPage() {
  if ($index.value === 0) {
    $index.value = yuRoleMes.value.length - 1
    return
  }
  $index.value--
}
// 下一页
function nextPage() {
  // 到最后一页
  if ($index.value === yuRoleMes.value.length - 1) {
    $index.value = 0
    return
  }
  $index.value++
}
// 点击操作
function handleRoleSwitchPC(index, e) {
  isCloseTranstion.value = false
  $index.value = index
}
//#endregion

//#region 移动端
const isPC = ref(false)
isPC.value = isPCSystem() //是否为PC
function isPCSystem() {
  var userAgentInfo = navigator.userAgent
  var Agents = ['Android', 'iPhone', 'SymbianOS', 'Windows Phone', 'iPad', 'iPod']
  var flag = true
  for (var v = 0; v < Agents.length; v++) {
    if (userAgentInfo.indexOf(Agents[v]) > 0) {
      flag = false
      break
    }
  }
  return flag
}

const isCloseTranstion = ref(false) //控制是否显示动画效果
const startX = ref(0) //记录开始位置
const endX = ref(0) //记录结束位置
const moveDistanceX = ref(0) //按下抬起滑动距离
const recordLastMove = ref(0) //记录上次滑动的距离，用于下次累加
const moveDistanceM = ref(0) //最终移动的距离
const moveDistanceMobile = computed(() => {
  return moveDistanceM.value + 'px'
})
// 触摸开始
function handleTouchStart(e) {
  isCloseTranstion.value = true // 开始移动 关闭动画
  startX.value = e.touches[0].pageX || e.changedTouches[0].pageX
}

// 触摸移动
function handleTouchMove(e) {
  e.preventDefault()
  isCloseTranstion.value = true // 开始移动 关闭动画
  moveDistanceX.value = (e.changedTouches[0].pageX || e.touches[0].pageX) - startX.value // 计算移动距离
  moveDistanceM.value = recordLastMove.value + moveDistanceX.value
}

// 触摸抬起
function handleTouchEnd(e) {
  e.preventDefault()
  // 抬起时开启动画
  isCloseTranstion.value = false
  // 计算结束距离
  endX.value = e.changedTouches[0].pageX || e.touches[0].pageX
  // 计算移动距离，判断应该上一页还是下一页，直接更改index即可在原先基础上整页移动
  moveDistanceX.value = endX.value - startX.value
  // 最后一栏的位置
  const rightMaxValue = (yuRoleMes.value.length - 5) * -64

  // 如果是点击滑动
  if (moveDistanceX.value === 0) {
    isClickMobile(e)
    return
  }

  // 如果照片小于五张或者直接往右边移动不进行位移移动
  if (yuRoleMes.value.length < 5 || moveDistanceM.value > 0) {
    moveDistanceM.value = 0
    recordLastMove.value = 0
    return
  }

  // // 如果直接往右滑动
  // if (moveDistanceM.value > 0) {
  //   moveDistanceM.value = 0
  //   recordLastMove.value = 0
  //   return
  // }
  //在最后一栏继续往左边滑动
  if (moveDistanceM.value < rightMaxValue) {
    moveDistanceM.value = rightMaxValue
    recordLastMove.value = rightMaxValue
    return
  }
  // 其他情况
  moveDistanceM.value = recordLastMove.value + Math.round(moveDistanceX.value / 64) * 64
  recordLastMove.value = Math.round(moveDistanceM.value / 64) * 64
}

function isClickMobile(e) {
  $index.value = Math.floor(e.target.offsetLeft / 64)
  // 是不是前三个
  const firstThree = $index.value < 3
  if (firstThree) {
    moveDistanceM.value = 0
    recordLastMove.value = 0
    return
  }

  // 是不是后三个
  const lastThree = $index.value > yuRoleMes.value.length - 4 //后三张往左移动
  if (lastThree) {
    moveDistanceM.value = (yuRoleMes.value.length - 5) * -64
    recordLastMove.value = (yuRoleMes.value.length - 5) * -64
    return
  }

  // 中间情况
  moveDistanceM.value = ($index.value - 2) * -64
  recordLastMove.value = ($index.value - 2) * -64
}
//#endregion
</script>
<template>
  <main>
    <!-- 底部操作 -->
    <div class="role-page" :style="{ height: isPC ? '190px' : '120px' }">
      <div class="role-outer" :style="{ width: isPC ? '830px' : '320px' }">
        <template v-if="isPC">
          <div class="left-arrow" :style="{ backgroundImage: `url(${leftSwitchArrow})` }" @click="lastPage"></div>
          <div class="right-arrow" :style="{ backgroundImage: `url(${rightSwitchArrow})` }" @click="nextPage"></div>
        </template>

        <!-- 角色容器 -->
        <div class="role-contener">
          <div
            class="role-inner"
            :class="{ activeTranstion: isCloseTranstion }"
            :style="{ left: isPC ? moveDistancePC : moveDistanceMobile }"
            ref="element"
            @touchstart="handleTouchStart"
            @touchend="handleTouchEnd"
            @touchmove="handleTouchMove"
          >
            <!-- 内层容器 PC-->
            <template v-if="isPC">
              <div
                v-for="(item, index) in yuRoleMes"
                class="role-item-pc"
                :style="{
                  backgroundPosition: $index === index ? '0 -132px' : '',
                  backgroundImage: `url(${bottomRoleBac})`,
                }"
                :key="index"
                @click="handleRoleSwitchPC(index)"
              >
                <img class="item-avater" :src="item.roleAvatar" />
                <p class="item-name" :style="{ color: $index === index ? '#000' : '#fff' }">{{ item.roleName }}</p>
              </div>
            </template>
            <!-- 内层容器 移动 -->
            <template v-else>
              <div
                v-for="(item, index) in yuRoleMes"
                class="role-item-mobile"
                :style="{
                  // backgroundPosition: $index === index ? '0 -132px' : '',
                  background: !isPC && $index === index ? `transparent  url(${mobileActiveBac}) round` : '',
                }"
                :key="index"
              >
                <img class="item-avater-mobile" :src="item.roleAvatar" />
                <p class="item-name-mobile" :style="{ color: $index === index ? '#000' : '#fff' }">
                  {{ item.roleName }}
                </p>
              </div>
            </template>
          </div>
        </div>
      </div>
    </div>
    <!-- 指示器 -->
    <ul class="aside">
      <li v-for="(item, index) in roleSidebar" @click="changeRole(index)">
        <span :class="{ active: index === currentIndex }"></span>
        <div v-show="index === currentIndex" class="show-dec">{{ item }}</div>
      </li>
    </ul>
    <!-- 角色介绍 -->
    <div class="role-introduce" :style="{ left: isPC ? '20%' : '2%' }">
      <div v-if="isPC" class="role-line"></div>
      <div class="introduce-name">{{ roleNameSwitch }}</div>
      <div v-if="isPC" class="introduce-info" v-html="roleIntroduce"></div>
    </div>
    <!-- 装饰背景 灰色斜背景 -->
    <div
      v-if="isPC"
      class="decorate-bac"
      :style="{
        backgroundImage: `url(https://ys.mihoyo.com/main/_nuxt/img/6c9d197.png)`,
      }"
    ></div>
    <!-- 装饰背景 花图案 -->
    <img
      v-if="isPC"
      class="decorate2-bac"
      src="https://uploadstatic.mihoyo.com/contentweb/20200220/2020022016441018411.png"
      alt=""
    />
    <!-- 背景-两张切换 -->
    <div class="background-wrapper">
      <!-- 第一张背景 -->
      <div
        class="role-background role-bg1"
        :style="{
          backgroundImage: outerTwoBackground[0],
        }"
      ></div>
      <!-- 第二张背景 -->
      <div
        class="role-background role-bg2"
        :style="{
          backgroundImage: outerTwoBackground[1],
        }"
      ></div>
    </div>
    <!-- 角色背景 -->
    <div class="role-wrapper">
      <div class="role-box">
        <img
          v-for="(item, index) in yuRoleMes"
          class="role-picture"
          :class="[index === $index ? 'show-background-pc' : 'hide-background']"
          :key="index"
          :src="item.roleBackground"
        />
      </div>
    </div>
  </main>
</template>
<style lang="scss" scoped>
main {
  // min-width: 1200px;
  position: relative;
  height: 100%;
  width: 100%;
  overflow: hidden;
}
.activeTranstion {
  transition: all 0ms ease 0s !important;
}
.decorate-bac {
  height: 100%;
  width: 100%;
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 2;
  background-position: center bottom;
}
.decorate2-bac {
  width: auto;
  height: 378px;
  position: absolute;
  top: 10%;
  left: 15%;
  z-index: 1;
  background-position: center bottom;
  opacity: 0.3;
}
// 人物介绍
.role-introduce {
  position: absolute;
  top: 15%;
  left: 20%;
  z-index: 11;
  .role-line {
    width: 180px;
    height: 4px;
    background-color: rgb(227, 188, 140);
    margin-bottom: 20px;
  }
  .introduce-name {
    font-size: 60px;
    color: #fff;
  }
  .introduce-info {
    width: 455px;
    background-color: #18212f;
    color: #fff;
    padding: 10px;
    border-radius: 6px;
    line-height: 27px;
    background: rgba(1, 1, 1, 0.5);
    margin-top: 20px;
    max-height: 130px;
    overflow-y: auto;
  }
}

// 底部操作
.role-page {
  width: 100%;
  height: 190px;
  background: rgba(204, 204, 204, 0.2);
  position: fixed;
  bottom: 0;
  z-index: 100;
  .role-outer {
    position: relative;
    width: 830px;
    height: 150px;
    margin: 0 auto;
    .left-arrow {
      left: -150px;
      top: calc(50% - 40px);
      position: absolute;
      background-size: cover;
      cursor: pointer;
      width: 45px;
      height: 64px;
    }

    .right-arrow {
      position: absolute;
      top: calc(50% - 40px);
      right: -150px;
      background-size: cover;
      cursor: pointer;
      width: 45px;
      height: 64px;
    }
    .role-contener {
      position: relative;
      overflow: hidden;
      margin-top: 20px;
      width: 100%;
      height: 100%;

      .role-inner {
        position: absolute;
        top: 0;
        left: 0;
        display: flex;
        transition: all ease-in-out 0.3s;
        .role-item-pc {
          position: relative;
          margin-right: 34px;
          width: 110px;
          height: 132px;
          .item-avater {
            display: block;
            margin: 2px auto 0;
            width: 105px;
            height: 105px;
          }
          .item-name {
            font-size: 16px;
            text-align: center;
            width: 100%;
            color: #fff;
            text-shadow: 0 2px 6px rgba(0, 0, 0, 0.2);
            line-height: 22px;
            margin: 0;
          }
          &:hover {
            cursor: pointer;
            background-position: 0 -132px;
          }
          &:hover .item-name {
            color: #000 !important;
          }
        }
        .role-item-mobile {
          width: 64px;
          height: 80px;
          display: flex;
          flex-direction: column;
          justify-content: center;
          align-items: center;

          .item-avater-mobile {
            width: 55px;
            height: 55px;
            padding-top: 8px;
          }
          .item-name-mobile {
            font-size: 14px;
            height: 17px;
            text-align: center;
            width: 100%;
            color: #fff;
            text-shadow: 0 2px 6px rgba(0, 0, 0, 0.2);
            margin: 0;
          }
        }
      }
    }
  }
}
// 背景角色
.role-wrapper {
  overflow: hidden;
  height: 100%;
  width: 100%;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  position: absolute;
  z-index: 10;
  .role-box {
    height: 100%;
    width: 100%;
    position: relative;
    .role-picture {
      position: absolute;
      height: 100%;
      // right: 0;
      // animation: bg-move 0.1s linear;
    }
    .show-background-pc {
      right: 0;
      opacity: 1;
      transition: all 0.3s;
    }
    .show-background-mobile {
      right: 445px;
      opacity: 1;
      transition: all 0.3s;
    }
    .hide-background {
      right: -1000px;
      opacity: 0;
      transition: all 0.3s;
    }
  }
}

@keyframes bg-move {
  0% {
    right: -2000px;
    opacity: 0;
  }
  50% {
    right: -1000px;
    opacity: 0.5;
  }
  60% {
    right: -800px;
    opacity: 0.6;
  }
  70% {
    right: -600px;
    opacity: 0.7;
  }
  80% {
    right: -400px;
    opacity: 0.8;
  }
  90% {
    right: -200px;
    opacity: 0.9;
  }
  100% {
    right: 0px;
    opacity: 1;
  }
}
// 两张背景照片切换
.role-background {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  height: 100%;
  width: 100%;
  background-position: center;
  background-size: cover;
  transform-origin: center;
}
// 第一张永远存在进行扩张
.role-bg1 {
  animation: breath 80s infinite linear;
  opacity: 1;
}
// 第二张也在扩张，只不过每15秒内进行显示隐藏
.role-bg2 {
  animation: bg-change 15s infinite linear, breath 80s infinite linear;
  opacity: 0;
}
// 用于第二张的显示隐藏
@keyframes bg-change {
  48% {
    opacity: 0;
  }

  50% {
    opacity: 1;
  }

  98% {
    opacity: 1;
  }

  100% {
    opacity: 0;
  }
}
// 用于向外扩张效果
@keyframes breath {
  0% {
    transform: scale(1);
  }

  50% {
    transform: scale(1.2);
  }

  100% {
    transform: scale(1);
  }
}
// 指示器样式
.active {
  display: inline-block;
  width: 16px !important;
  height: 16px !important;
}
.aside {
  list-style: none;
  position: fixed;
  left: 20px;
  top: 50%;
  z-index: 100;
  transform: translateY(-50%);
  margin: 0;
  padding: 0;
  li {
    height: 20px;
    width: 20px;
    margin: 10px;
    display: flex;
    align-items: center;
    justify-content: center;
    position: relative;
    &:hover .show-dec {
      display: block !important;
    }
    .show-dec {
      text-align: left;
      position: absolute;
      width: 80px;
      left: 24px;
      padding: 1px;
      margin-right: 5px;
      color: #000;
      transition: all linear 0.1s;
      font-size: 15px;
      background-color: #fff;
    }
    span {
      display: inline-block;
      border-radius: 100%;
      border: #fff solid 1px;
      width: 8px;
      height: 8px;
      display: inline-block;
      background-color: #fff;
      transition: all ease-in-out 0.2s;
    }
    &:hover span {
      width: 16px;
      height: 16px;
      background-color: #fff;
      cursor: pointer;
    }
  }
}
/* 自定义滚动条样式 */

::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

::-webkit-scrollbar-track {
  background-color: #f1f1f1;
  border-radius: 2px;
}

::-webkit-scrollbar-thumb {
  background-color: #888;
  border-radius: 2px;
}

::-webkit-scrollbar-thumb:hover {
  background-color: #555;
}
</style>
