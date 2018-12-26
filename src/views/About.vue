<template>
  <div class="person-svg">
    <van-button type="primary" @click="turn()">正反转换</van-button>
    <!-- 人体svg热区图  -->
    <svg id="full-body" x="0px" y="0px" viewBox="0 0 310 385" style="width:100%" v-bind:preserveAspectRatio="preserveAspectRatio?'xMidYMid slice':''"
      xml:space="preserve" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
      <image v-bind:xlink:href="imgPath" height="100%" width="100%" x="0" y="0"></image>
      <path v-for="(value,index) in pathList" :key="index"
        class="layers layer-show"
        fill="#27BFCF" 
        fill-rule="evenodd" 
        opacity=".3" 
        v-bind:d="value.d" 
        v-bind:transform="value.transform"
        @click="getLevelTwo('HEAD_1',1)">
      </path>
    </svg>

    <!-- 人体部位层级数据 -->
    <van-popup v-model="show" position="bottom" :overlay="false" :lock-scroll="false">
      
      <div class="popup-cell">
        <div class="popup-cell-title">请选择您的症状</div>
        <van-icon name="cross" color="#fff" @click="hideMenu"></van-icon>
      </div>
      
      <!-- 二级菜单 -->
      <div v-if="!showNext" v-for="(value,index) in levelTwo" :key="index">
        <div class="cell-link van-hairline--bottom" @click="getLevelThree(value.code)">
          <div class="popup-cell-title">{{value.name}}</div>
          <van-icon name="arrow" color="#333" size="14px"></van-icon>
        </div>
      </div>

      <!-- 返回二级菜单 -->
      <div v-if="showNext" class="level-one" @click="backLevel"><van-icon name="arrow-left" color="#1e84d8" size="16px"></van-icon>返回</div>
      <!-- 三级菜单 -->
      <div v-if="showNext" v-for="(item,indexs) in levelThree" :key="indexs+'-3'">
        <div class="cell-link van-hairline--bottom">
          <div class="popup-cell-title">{{item.name}}</div>
          <van-icon name="arrow" color="#333" size="14px"></van-icon>
        </div>
      </div>

    </van-popup>

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
      showNext: false,
      sexType: "male",
      faceType:'front',
      preserveAspectRatio: false,
      pathList: [],
      levelTwo:[],
      levelThree:[],
      imgPath: require('../assets/male-f.png')
    };
  },

  mounted() {
    this.sexType = this.$route.query.sexType
    this.faceType = 'front'
    if(this.$route.query.sexType == 'male'){
      this.imgPath = require('../assets/male-f.png')
      this.pathList = svgPath.male.front.levelOne;
    }else{
      this.imgPath = require('../assets/female-f.png')
      this.pathList = svgPath.female.front.levelOne;
    }
  },

  methods: {
    hideMenu() {
      this.show = false
      this.showNext = false
      this.preserveAspectRatio = false
    },
    getLevelTwo(paramName,level) {
      if(level == 1){
        let paths = svgPath[this.sexType][this.faceType]
        this.pathList = paths.levelTwo
        console.log(paths.levelTwo)
        // this.preserveAspectRatio = true
      }else{
        fetch("http://cm.changrentech.com:8092/bodyparts.do?parentCode="+paramName)
        .then(response => response.json())
        .then(res => {
          this.show = true
          this.levelTwo = res.result
        })
        .catch(err => {
          this.$toast({message:'检测异常，稍后再试',mask:false,duration:1500})
          console.log(err);
        });
      }
    },
    getLevelThree(paramName){
      fetch("http://cm.changrentech.com:8092/symptoms/symptomsList?code="+paramName+"&iscommon=false")
        .then(response => response.json())
        .then(res => {
          console.log(res.result)
          this.levelThree = res.result
        })
        .catch(err => {
          this.$toast({message:'检测异常，稍后再试',mask:false,duration:1500})
          console.log(err);
        });
    },
    backLevel(){
      this.showNext = false
    },
    turn(){
      this.show = false
      let sexType = this.sexType
      let faceType = this.faceType
      if(faceType == 'front'){
        this.faceType = 'back'
        this.imgPath = require('../assets/'+sexType+'-b.png')
        let pathf = svgPath[sexType][this.faceType]
        this.pathList = pathf.levelOne
      }else{
        this.faceType = 'front'
        this.imgPath = require('../assets/'+sexType+'-f.png')
        let pathb = svgPath[sexType][this.faceType]
        this.pathList = pathb.levelOne
      }
    }
  }
};
</script>
<style lang="scss">
.person-svg{
  width:100vw;
  height:100vh;
}
#full-body {
  display: block;
  height: 96%;
  transition: width .5s;
  -webkit-transition: width .5s;
  -moz-transition: width .5s;
  .layers {
    cursor: pointer;
    display: none;
    opacity: 0;
    outline: 0;
    transition: opacity .5s linear;
    &:hover {
      opacity: .525;
      outline: 0;
      transition: opacity .5s linear;
    }
  }
  .layer-show{
    display:block;
  }
}

.van-popup {
  height: 30%;
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
.cell-link {
  @extend .popup-cell;
  background: #ffffff;
  .popup-cell-title {
    font-size: 14px;
    color: #333;
  }
}
</style>
