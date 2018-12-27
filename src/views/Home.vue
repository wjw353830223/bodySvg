<template>
  <div class="person-svg">
    <!-- <van-button type="primary" @click="turn()">正反转换</van-button> -->
    <div style="position:absolute;top:20px;right:20px;background:#fff;width:30px;height:30px;line-height:30px;text-align:center;border-radius:5px;">
      <van-icon name="exchange" color="#f34c4c" @click="turn" size="20px"></van-icon>
    </div>
    <!-- 人体svg热区图  -->
    <svg
      id="full-body"
      x="0px"
      y="0px"
      viewBox="0 0 310 385"
      style="width:100%"
      v-bind:preserveAspectRatio="preserveAspectRatio?'xMidYMid slice':''"
      xml:space="preserve"
      xmlns="http://www.w3.org/2000/svg"
      xmlns:xlink="http://www.w3.org/1999/xlink"
    >
      <image v-bind:xlink:href="imgPath" height="100%" width="100%" x="0" y="0"></image>
      <path
        v-for="(value,index) in pathList"
        :key="index"
        class="layers layer-show"
        fill="#f34c4c"
        fill-rule="evenodd"
        opacity=".85"
        v-bind:d="value.d"
        v-bind:transform="value.transform"
        @click="getLevelTwo('HEAD_1',value.level)"
      ></path>
    </svg>

    <!-- 人体部位层级数据 -->
    <van-popup v-model="show" position="bottom" :overlay="false" :lock-scroll="false" ref="popup">
      <div class="popup-cell">
        <div class="popup-cell-title">请选择您的症状</div>
        <van-icon name="cross" color="#fff" @click="hideMenu"></van-icon>
      </div>

      <div class="popup-body">
        <!-- 二级菜单 -->
        <div v-if="!showNext" v-for="(value,index) in levelTwo" :key="index">
          <div class="cell-link van-hairline--bottom" @click="getLevelThree(value.code)">
            <div class="popup-cell-title">{{value.name}}</div>
            <van-icon name="arrow" color="#333" size="14px"></van-icon>
          </div>
        </div>

        <!-- 返回二级菜单 -->
        <div v-if="showNext" class="level-one" @click="backLevel">
          <van-icon name="arrow-left" color="#1e84d8" size="16px"></van-icon>返回
        </div>
        <!-- 三级菜单 -->
        <van-checkbox-group v-model="checkResult" v-if="showNext">
          <van-cell-group>
            <van-cell
              v-for="(item,indexs) in levelThree"
              clickable
              :key="indexs"
              :title="item.name"
              @click="toggle(indexs)"
            >
              <van-checkbox :name="item" ref="checkboxes"/>
            </van-cell>
          </van-cell-group>
        </van-checkbox-group>
        <button class="next-but" v-if="showNext">下一步</button>
      </div>
    </van-popup>
  </div>
</template>
<script>
import svgPath from "../../public/json/svgPath";
import {
  Popup,
  Toast,
  Icon,
  Checkbox,
  CheckboxGroup,
  CellGroup,
  Cell
} from "vant";
export default {
  components: {
    [Cell.name]: Cell,
    [Icon.name]: Icon,
    [Toast.name]: Toast,
    [Popup.name]: Popup,
    [Checkbox.name]: Checkbox,
    [CellGroup.name]: CellGroup,
    [CheckboxGroup.name]: CheckboxGroup
  },

  data() {
    return {
      show: false,
      showNext: false,
      sexType: "male",
      faceType: "front",
      preserveAspectRatio: false,
      pathList: [],
      levelTwo: [],
      levelThree: [],
      checkResult: [],
      imgPath: require("../assets/male-f.png")
    };
  },

  mounted() {
    this.faceType = "front";

    if (this.$route.query.sexType == "male" || this.$route.query.sexType == undefined) {
      this.sexType = this.$route.query.sexType;
      this.imgPath = require("../assets/male-f.png");
      this.pathList = svgPath.male.front.levelOne;
    } else {
      this.sexType = this.$route.query.sexType;
      this.imgPath = require("../assets/female-f.png");
      this.pathList = svgPath.female.front.levelOne;
    }

  },

  methods: {
    toggle(index) {
      this.$refs.checkboxes[index].toggle();
    },
    hideMenu() {
      this.show = false;
      this.showNext = false;
      this.preserveAspectRatio = false;
    },
    getLevelTwo(paramName, level) {
      console.log(this.$refs.popup.$el);
      this.showNext = false;
      fetch(
        "http://cm.changrentech.com:8092/bodyparts.do?parentCode=" + paramName
      )
        .then(response => response.json())
        .then(res => {
          this.show = true;
          this.levelTwo = res.result;
        })
        .catch(err => {
          this.$toast({
            message: "检测异常，稍后再试",
            mask: false,
            duration: 1500
          });
          console.log(err);
        });
    },
    getLevelThree(paramName) {
      fetch(
        "http://cm.changrentech.com:8092/symptoms/symptomsList?code=" +
          paramName +
          "&iscommon=false"
      )
        .then(response => response.json())
        .then(res => {
          console.log(res.result);
          this.levelThree = res.result;
          this.showNext = true;
        })
        .catch(err => {
          this.$toast({
            message: "检测异常，稍后再试",
            mask: false,
            duration: 1500
          });
          console.log(err);
        });
    },
    backLevel() {
      this.showNext = false;
    },
    turn() {
      this.show = false;
      this.showNext = false;
      let sexType = this.sexType;
      let faceType = this.faceType;
      if (faceType == "front") {
        this.faceType = "back";
        this.imgPath = require("../assets/" + sexType + "-b.png");
        let pathf = svgPath[sexType][this.faceType];
        this.pathList = pathf.levelOne;
      } else {
        this.faceType = "front";
        this.imgPath = require("../assets/" + sexType + "-f.png");
        let pathb = svgPath[sexType][this.faceType];
        this.pathList = pathb.levelOne;
      }
    }
  }
};
</script>
<style lang="scss">
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

.person-svg {
  width: 100vw;
  height: 100vh;
  background: url("../assets/svgBg.png");
  background-size: cover;
}
#full-body {
  display: block;
  height: 96%;
  transition: width 0.5s;
  -webkit-transition: width 0.5s;
  -moz-transition: width 0.5s;
  .layers {
    cursor: pointer;
    display: none;
    opacity: 0;
    outline: 0;
    transition: opacity 0.5s linear;
    &:hover {
      opacity: 0.525;
      outline: 0;
      transition: opacity 0.5s linear;
    }
  }
  .layer-show {
    display: block;
  }
}

.van-popup {
  height: 30%;
  // overflow-y: hidden;
  .popup-body {
    overflow-y: auto;
    // height: 100%;
  }
}
.level-one {
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
.cell-link {
  @extend .popup-cell;
  background: #ffffff;
  .popup-cell-title {
    font-size: 14px;
    color: #333;
  }
}
.van-cell__title {
  flex-basis: 90%;
  display: flex;
  justify-content: space-between;
}
.van-cell__value {
  flex-basis: 10%;
}

.next-but {
  width: 80%;
  height: 45px;
  line-height: 45px;
  text-align: center;
  background: #1e84d8;
  color: #fff;
  border: 1px solid #1e84d8;
  box-sizing: border-box;
  border-radius: 8px;
  font-size: 16px;
  margin: 10px auto;
}
</style>
