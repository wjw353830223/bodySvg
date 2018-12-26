<template>
  <div>
    <van-button type="primary" size="normal" @click="showMenu">显示</van-button>
    <van-popup v-model="show" position="bottom" :overlay="false">
      
      <div class="popup-cell popup-cell-fixed">
        <div class="popup-cell-title">要显示的字体</div>
        <van-icon name="cross" color="#fff" @click="hideMenu"></van-icon>
      </div>
      
      <!-- 一级菜单 -->
      <div v-if="!showNext" v-for="(value,index) in levelOne" :key="index">
        <div class="cell-link van-hairline--bottom" @click="getLevelTwo(value.code)">
          <div class="popup-cell-title">{{value.name}}</div>
          <van-icon name="arrow" color="#333" size="14px"></van-icon>
        </div>
      </div>

      <!-- 返回一级菜单 -->
      <div v-if="showNext" class="level-one" @click="backLevelOne"><van-icon name="arrow-left" color="#1e84d8" size="16px"></van-icon>返回</div>
      <!-- 二级菜单 -->
      <div v-if="showNext" v-for="(item,indexs) in levelTwo" :key="indexs+'-2'">
        <div class="cell-link van-hairline--bottom" @click="getLevelTwo(item.code)">
          <div class="popup-cell-title">{{item.name}}</div>
          <van-icon name="arrow" color="#333" size="14px"></van-icon>
        </div>
      </div>

    </van-popup>
    <svg>
      <path v-for="(value,index) in frontOne" :key="index"></path>
    </svg>
  </div>
</template>
<script>
import svgPath from "../../public/json/svgPath";
import { Popup, Toast, Button, Icon } from "vant";
export default {
  components: {
    [Icon.name]: Icon,
    [Toast.name]: Toast,
    [Popup.name]: Popup,
    [Button.name]: Button
  },

  data() {
    return {
      show: false,
      showNext:false,
      sexType: "male",
      frontOne: [],
      frontTwo: [],
      backOne: [],
      backTwo: [],
      levelOne: [],
      levelTwo:[]
    };
  },

  mounted() {
    this.frontOne = svgPath.male.front.levelOne;
  },

  methods: {
    showMenu() {
      this.show = true;
      fetch("http://cm.changrentech.com:8092/bodyparts.do?parentCode=l1")
        .then(response => response.json())
        .then(res => {
          this.levelOne = res.result
        })
        .catch(err => {
          console.log(err);
        });
    },
    hideMenu() {
      this.show = false;
      this.showNext = false
    },
    getLevelTwo(param) {
      fetch("http://cm.changrentech.com:8092/bodyparts.do?parentCode="+param)
        .then(response => response.json())
        .then(res => {
          console.log(res.result)
          this.levelTwo = res.result
          this.showNext = true
        })
        .catch(err => {
          console.log(err);
        });
    },
    backLevelOne(){
      this.showNext = false
    }
  }
};
</script>
<style lang="scss">
.van-popup {
  height: 50%;
}
.level-one{
  font-size: 16px;
  color: #1e84d8;
  line-height: 45px;
  display: flex;
  flex-direction: row;
  justify-content: flex-start;
  align-items: center;
  box-sizing: border-box;
  padding-left: 20px;
}
.popup-cell {
  width: 100%;
  height: 45px;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  padding: 0 20px;
  background: #1e84d8;
  box-sizing: border-box;
  .popup-cell-title {
    font-size: 16px;
    color: #fff;
  }
}
.popup-cell-fixed{
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 2000;
}
.cell-link {
  @extend .popup-cell;
  background: #ffffff;
  .popup-cell-title {
    font-size: 14px;
    color: #333;
  }
}
</style>
