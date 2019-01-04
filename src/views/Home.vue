<template>
  <div class="person-svg">
    <div class="turn-face" @click="turn">
      <img src="../assets/img/turn.png">
    </div>
    <!-- 人体svg热区图  -->
    <svg
      id="full-body"
      x="0px"
      y="0px"
      viewBox="0 0 310 385"
      style="width:100%"
      v-bind:preserveAspectRatio="preserveAspectRatio?'xMidYMax slice':''"
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
        @click="getLevelTwo(value.code,value.level)"
      ></path>
    </svg>

    <!-- 人体部位层级数据 -->
    <van-popup v-model="show" position="bottom" :overlay="false" :lock-scroll="false">
      <div class="popup-cell">
        <div class="popup-cell-title">请选择您的症状</div>
        <van-icon name="cross" color="#fff" @click="hideMenu"></van-icon>
      </div>

      <div class="popup-body" v-bind:style="{height:clientHeight*0.3-40+'px'}" ref="popupBody">
        <div v-if="!showNext && levelTwo.length == 0" class="data-null">
          <img src="../assets/img/data-null.png">
        </div>
        <!-- 二级菜单 -->
        <div v-if="!showNext && levelTwo.length > 0 " v-for="(value,index) in levelTwo" :key="index">
          <div class="cell-link van-hairline--bottom" @click="getLevelThree(value.code)">
            <div class="popup-cell-title">{{value.name}}</div>
            <van-icon name="arrow" color="#333" size="14px"></van-icon>
          </div>
        </div>

        <!-- 返回二级菜单 -->
        <div v-if="showNext" class="level-one" @click="backLevel">
          <van-icon name="arrow-left" color="#1e84d8" size="14px"></van-icon>返回上级
        </div>
        <div v-if="showNext && levelThree.length == 0" class="data-null">
          <img src="../assets/img/data-null.png">
        </div>
        <div v-if="showNext && levelThree.length > 0">
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
              <van-checkbox :name="item" checked-color="#0E7BC3" shape="square" ref="checkboxes"/>
            </van-cell>
          </van-cell-group>
        </van-checkbox-group>
        <button class="next-but" v-if="levelThree.length>0" @click="nextSearch">下一步</button>
        </div>
        
      </div>
    </van-popup>
  </div>
</template>
<script>
import svgPath from ".././assets/json/svgPath";
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
      clientHeight: 0,
      preserveAspectRatio: false,
      pathList: [],
      levelTwo: [],
      levelThree: [],
      checkResult: [],
      imgPath: require("../assets/img/male-f.png")
    };
  },

  mounted() {
    this.faceType = "front";
    this.clientHeight = this.$el.clientHeight
    if (
      this.$route.query.gender == "male" ||
      this.$route.query.gender == undefined
    ) {
      this.sexType = "male";
      this.imgPath = require("../assets/img/male-f.png");
      this.pathList = svgPath.male.front.levelOne;
    } else {
      this.sexType = "female";
      this.imgPath = require("../assets/img/female-f.png");
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
      this.showNext = false;
      fetch("http://39.96.15.44:8092/bodyparts.do?parentCode=" + paramName)
        .then(response => response.json())
        .then(res => {
          this.show = true;
          this.levelTwo = res.result;
          this.preserveAspectRatio = true;
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
      let sexType = this.sexType
      let faceType = this.faceType
      this.$refs.popupBody.scrollTop = 0
      fetch(
        "http://39.96.15.44:8092/symptoms/symptomsList?bodypartName="+paramName+"&sex="+sexType+"&isfront="+faceType+"&iscommon=true"
      )
        .then(response => response.json())
        .then(res => {
          this.showNext = true;
          this.levelThree = res.result;
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
      this.preserveAspectRatio = false;
      let sexType = this.sexType;
      let faceType = this.faceType;
      if (faceType == "front") {
        this.faceType = "back";
        this.imgPath = require("../assets/img/" + sexType + "-b.png");
        let pathf = svgPath[sexType][this.faceType];
        this.pathList = pathf.levelOne;
      } else {
        this.faceType = "front";
        this.imgPath = require("../assets/img/" + sexType + "-f.png");
        let pathb = svgPath[sexType][this.faceType];
        this.pathList = pathb.levelOne;
      }
    },
    nextSearch() {
      let andParams = {};
      andParams["data"] = this.checkResult;
      let iosParams = {
        "method": "symptomSelect",
        "params": andParams
      }
      var ua = navigator.userAgent.toLowerCase();
      if (/iphone|ipad|ipod/.test(ua)) {
        window.webkit.messageHandlers.utils.postMessage(JSON.stringify(iosParams));
      } else if (/android/.test(ua)) {
        window.jsNativeObj.getSymptomsSelected(JSON.stringify(andParams))
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
  position: relative;
}
.turn-face {
  position: fixed;
  top: 108px;
  right: 20px;
  background: #02355F;
  width: 35px;
  height: 35px;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 5px;
  z-index:1000;
  img{
    width: 21px;
    height: 21px;
  }
}
.person-svg {
  width: 100vw;
  height: 100vh;
  background: url("../assets/img/svgBg.png");
  background-size: cover;
  position: relative;
}
#full-body {
  display: block;
  height: 100%;
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
  overflow-y: hidden;
  .popup-body {
    overflow-y: auto;
    height: 100%;
  }
}
.level-one {
  font-size: 14px;
  color: #0E7BC3;
  line-height: 40px;
  display: flex;
  flex-direction: row;
  justify-content: flex-start;
  align-items: center;
  box-sizing: border-box;
  padding-left: 20px;
}
.popup-cell {
  width: 100%;
  height: 40px;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  padding: 0 20px;
  background: #0E7BC3;
  box-sizing: border-box;
  .popup-cell-title {
    font-size: 14px;
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
  height: 40px;
  line-height: 40px;
  text-align: center;
  background: #0E7BC3;
  color: #fff;
  border: 1px solid #0E7BC3;
  box-sizing: border-box;
  border-radius: 8px;
  font-size: 16px;
  margin: 10px auto;
}

.data-null{
  width: 100%;
  img{
    margin-top: 30px;
    width: 53px;
    height: 49px;
  }
}
</style>
