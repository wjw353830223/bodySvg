<template>
  <div class='body-image' id='body-image'>
    <!-- 人体svg热区图  -->
    <svg id="full-body" x="0px" y="0px" v-bind:viewBox="svg.viewBox" v-bind:preserveAspectRatio="svg.preserveAspectRatio?'xMidYMid meet':''"
      xml:space="preserve" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
      <image v-bind:xlink:href="svg.img" height="100%" width="100%" x="0" y="0"></image>
      <path v-for="(value,index) in svg.pathes" :key="index"
        v-bind:class="value.class"   class='layer-show'
        v-bind:transform="value.transform"
        v-bind:svg-tooltip="value.svgTooltip"
        v-bind:d="value.d"
        v-bind:fill="value.fill"
        v-bind:fill-rule="value.fillRule"
        v-bind:opacity="value.opacity"
        @click="getHigherLever(value)">
      </path>
    </svg>
    <div class='contro'>
      <span @click="changeSvg('sex')"><i class='iconfont icon-xingbie'></i><font>性别切换</font></span>
      <span @click="changeSvg('deriction')"><i class='iconfont icon-fanzhuan'></i><font>前后翻转</font></span>
      <span @click="showAllSympptom"><i class='iconfont icon-caidan'></i><font>全部病症</font></span>
      <span @click="choosedSymptomShow=true"><i class='iconfont icon-liebiaochakan_'></i><font>已选病症</font></span>
    </div>
    <!-- 点击二级映射弹层 -->
    <van-popup v-model="levelThreeMap" position="bottom" :overlay="false" :lock-scroll="false" ref="popup-map">
      <div class="popup-cell">
        <div class="popup-cell-title">{{popupTitle}}</div>
        <van-icon name="cross" color="#fff" @click="hideMapMenu"></van-icon>
      </div>
      <van-checkbox-group v-model="levelThreeMapChecked">
        <van-cell-group>
          <van-cell
            v-for="(item,indexs) in levelThree"
            clickable
            :key="indexs"
            :title="item.Name"
          >
            <van-checkbox :name="item" ref="checkboxes-map"/>
          </van-cell>
        </van-cell-group>
      </van-checkbox-group>
      <button class="next-but" @click="addSymptom">添加</button>
    </van-popup>
    <!-- 已选病症弹层 -->
    <van-popup v-model="choosedSymptomShow" position="bottom" :overlay="false" :lock-scroll="false" ref="choosed-symptom-show">
      <div class="popup-cell">
        <div class="popup-cell-title">已选病症</div>
        <van-icon name="cross" color="#fff" @click="choosedSymptomShow=false"></van-icon>
      </div>
      <van-checkbox-group v-model="choosedSymptom">
          <van-cell-group>
            <van-cell
              v-for="(item,indexs) in choosedSymptom"
              clickable
              :key="indexs"
              :title="item.Name"
            >
              <van-checkbox :name="item.Name" v-model="item.checked"/>
            </van-cell>
          </van-cell-group>
        </van-checkbox-group>
    </van-popup>
    <!-- 全部病症弹层 -->
    <van-popup v-model='allSymptomShow' position="bottom" :overlay="false" :lock-scroll="false" ref="all-symptom-show">
      <div class="popup-cell">
        <div class="popup-cell-title">请选择您的症状</div>
        <van-icon name="cross" color="#fff" @click="allSymptomShow=false"></van-icon>
      </div>
      <div class="popup-body">
        <!-- 一级菜单 -->
        <div v-if="showLevelOne">
          <div v-for="(value,index) in getLevelOneData" :key="index">
            <div class="cell-link van-hairline--bottom" @click="getLeverTwo(value)">
              <div class="popup-cell-title">{{value.name}}</div>
              <van-icon name="arrow" color="#333" size="14px"></van-icon>
            </div>
          </div>
        </div>
        <!-- 二级菜单 -->
        <div v-if="showLevelTwo" class="level-one" @click="backLevelOne">
          <van-icon name="arrow-left" color="#1e84d8" size="16px"></van-icon>返回上级
        </div>
        <div v-if="showLevelTwo">
          <van-checkbox-group v-model="levelThreeDataChecked" v-if="currentLevelOneValue.name=='skin' || currentLevelOneValue.name=='general'">
            <van-cell-group>
              <van-cell
                v-for="(item,indexs) in levelThreeData"
                clickable
                :key="indexs"
                :title="item.Name"
              >
                <van-checkbox :name="item" ref="checkboxes-three"/>
              </van-cell>
            </van-cell-group>
          </van-checkbox-group>
          <button class="next-but" v-if="currentLevelOneValue.name=='skin' || currentLevelOneValue.name=='general'" @click="addSymptom">添加</button>
          <van-cell-group v-else>
            <van-cell
              v-for="(value,index) in getLevelTwoData"
              clickable
              :key="index"
              :title="value.name"
              @click="getLeverThree(value)"
            >
            <van-icon name="arrow" color="#333" size="14px"></van-icon>
            </van-cell>
          </van-cell-group>
        </div>
        <!-- 三级菜单 -->
        <div v-if="showLevelThree" class="level-one" @click="backLevelTwo">
          <van-icon name="arrow-left" color="#1e84d8" size="16px"></van-icon>返回上级
        </div>
        <van-checkbox-group v-model="levelThreeDataChecked" v-if="showLevelThree">
          <van-cell-group>
            <van-cell
              v-for="(item,indexs) in levelThreeData"
              clickable
              :key="indexs"
              :title="item.Name"
            >
              <van-checkbox :name="item" ref="checkboxes-three"/>
            </van-cell>
          </van-cell-group>
        </van-checkbox-group>
        <button class="next-but" v-if="showLevelThree" @click="addSymptom">添加</button>
      </div>
    </van-popup>
  </div>
</template>
<script>
import svgPath from "../../public/json/svgPathNew";
import { Popup, Toast, Button, Icon, Cell, Checkbox, CellGroup, CheckboxGroup, Dialog } from "vant";
export default {
  name:'Person',
  components: {
    [Cell.name]: Cell,
    [Icon.name]: Icon,
    [Toast.name]: Toast,
    [Popup.name]: Popup,
    [Checkbox.name]: Checkbox,
    [CellGroup.name]: CellGroup,
    [CheckboxGroup.name]: CheckboxGroup,
    [Dialog.name]: Dialog
  },
  data() {
    return {
      show: false,
      showNext: false,
      checkResult:false,
      levelTwo:[],
      levelThree:[],
      popupTitle:'请选择身体热区',
      levelThreeMap:false,
      svg:{
        sex: "male",
        deriction:'front',
        pathes: [{
          class:'',
          transform:'',
          svgTooltip:'',
          d:'',
          fill:'',
          fillRule:'',
          opacity:''
        }],
        img: '',
        viewBox:"0 0 310 385",
        preserveAspectRatio: false,
      },
      choosedSymptom:[],
      choosedSymptomShow:false,
      allSymptomShow:false,
      showLevelTwo:false,
      showLevelOne:true,
      showLevelThree:false,
      currentLevelOneValue:{},
      currentLevelTwoValue:{},
      levelTwoData:[],
      levelThreeData:[],
      levelThreeDataChecked:[],
      levelThreeMapChecked:[]
    };
  },
  mounted() {
    let sex='male';
    if(this.$route.query.sexType){
      sex=this.$route.query.sexType
    }
    let deriction='front'
    if(this.$route.query.deriction){
      deriction=this.$route.query.deriction
    }
    this.svg=this.initSvg(sex,deriction,this.svg.viewBox,this.svg.preserveAspectRatio)
  },
  methods: {
    showAllSympptom(){
      this.allSymptomShow=true;
      this.showLevelOne=true;
    },
    //初始化svg相关参数
    initSvg(sex,deriction,viewBox,preserveAspectRatio){
      let svg={};
      let pathes=svgPath[sex][deriction].levelOne
      svg.pathes=pathes
      svg.deriction=deriction
      svg.sex=sex
      if(deriction=='front'){
        svg.img=require('../assets/'+sex+'-f.png')
      }else{
        svg.img=require('../assets/'+sex+'-b.png')
      }
      svg.viewBox=viewBox
      svg.preserveAspectRatio=preserveAspectRatio
      return svg;
    },
    changeSvg(type){
      let sex='male'
      let deriction='front'
      if(type=='sex'){
        deriction = this.svg.deriction
        if(this.svg.sex=='male'){
          sex='female'
        }
        this.svg.sex = sex
      }
      if(type=='deriction'){
        sex=this.svg.sex
        if(this.svg.deriction=='front'){
          deriction='back'
        }
        this.svg.deriction = deriction
      } 
      if(this.svg.preserveAspectRatio){
        this.svg.viewBox='0 0 190 385';
        if(this.svg.sex=='male'){
          this.svg.viewBox='0 0 190 345';
        }
      }
      this.svg=this.initSvg(sex,deriction,this.svg.viewBox,this.svg.preserveAspectRatio)
    },
    hideMapMenu() {
      this.svg.preserveAspectRatio = false
      this.svg.pathes = svgPath[this.svg.sex][this.svg.deriction].levelOne
      this.svg.viewBox="0 0 310 385";
      this.levelThreeMap=false;
      this.allSymptomShow=false;
    },
    hideMenu() {
      this.show = false
      this.svg.preserveAspectRatio = false
      this.svg.pathes = svgPath[this.svg.sex][this.svg.deriction].levelOne
      this.svg.viewBox="0 0 310 385";
    },
    getHigherLever(value){
      let _self=this;
      if(value.level==1){
        this.svg.preserveAspectRatio = true;
        this.svg.viewBox='0 0 190 385';
        if(this.svg.sex=='male'){
          this.svg.viewBox='0 0 190 345';
        }
        setTimeout(function(){
          let levelTwo = svgPath[_self.svg.sex][_self.svg.deriction].levelTwo
          _self.svg.pathes = levelTwo;
          _self.levelTwo = levelTwo.filter(function(item){
              return item.parentName==value.name;
          })
        },500)
      }
      if(value.level==2){
        this.levelThreeMap=true;
        let _self=this
        setTimeout(function(){
          let levelThreeNew=_self.getThreeLevelData(value)
          _self.levelThree=levelThreeNew
        },500)
      }
    },
    //合并两个数组，去重
    concat_(arr1,arr2){
      var arr = arr1.concat();
      for(var i=0;i<arr2.length;i++){
        arr.indexOf(arr2[i]) === -1 ? arr.push(arr2[i]) : 0;
      }
      return arr;
    },
    addSymptom(){
      let choosedSymptom=this.concat_(this.levelThreeMapChecked,this.levelThreeDataChecked)
      this.choosedSymptom=choosedSymptom
      Dialog.confirm({
        title: '添加成功',
        message: '病症已添加到已选病症列表',
        confirmButtonText:'继续添加',
        cancelButtonText:'取消'
      }).then(() => {
        // on confirm
      }).catch(() => {
        this.showLevelThree=false;
        this.showLevelTwo=false;
        this.showLevelOne=false;
        this.hideMapMenu();
      });
    },
    getLeverTwo(value){
      this.currentLevelOneValue=value;
      this.showLevelOne=false;
      this.showLevelTwo=true;
      if(value.name=='skin' || value.name=='general'){
        this.levelThreeData=this.getSkinOrGeneralData(value)
      }else{
        let levelTwo = svgPath[this.svg.sex][this.svg.deriction].levelTwo
        this.levelTwo = levelTwo.filter(function(item){
            return item.parentName==value.name;
        })
      }
    },
    backLevelOne(){
      this.showLevelTwo=false;
      this.showLevelOne=true;
    },
    backLevelTwo(){
      this.showLevelThree=false;
      this.showLevelTwo=true;
    },
    getLeverThree(value){
      this.showLevelThree=true;
      this.showLevelTwo=false;
      this.showLevelOne=false;
      this.currentLevelTwoValue=value;
      this.levelThreeData=this.getThreeLevelData(this.currentLevelTwoValue)
    },
    getSkinOrGeneralData(value){
      let dir=''
      if(this.svg.sex=='male'){
        if(this.svg.deriction=='front'){
          dir = 'maleZ/'
        }else{
          dir = 'maleF/'
        }
      }else{
        if(this.svg.deriction=='front'){
          dir = 'feMaleZ/'
        }else{
          dir = 'feMaleF/'
        }
      }
      let levelThree=[];
      levelThree=require('public/json/' + dir + value.name+'.json')
      let levelThreeNew=[];
      levelThree.map(function(item){
        if(item.Name){
          item.checked=false;
          levelThreeNew.push(item)
        }
      })
      levelThreeNew.sort(function(a,b){
        return a.Name.toLowerCase()>b.Name.toLowerCase()
      })
      return levelThreeNew;
    },
    getThreeLevelData(value){
      let dir=''
      if(this.svg.sex=='male'){
        if(this.svg.deriction=='front'){
          dir = 'maleZ/'
        }else{
          dir = 'maleF/'
        }
      }else{
        if(this.svg.deriction=='front'){
          dir = 'feMaleZ/'
        }else{
          dir = 'feMaleF/'
        }
      }
      let levelThree=[];
      levelThree=require('public/json/' + dir + value.parentName+'/'+value.name+'.json')
      let levelThreeNew=[];
      levelThree.map(function(item){
        if(item.Name){
          item.checked=false;
          levelThreeNew.push(item)
        }
      })
      levelThreeNew.sort(function(a,b){
        return a.Name.toLowerCase()>b.Name.toLowerCase()
      })
      return levelThreeNew;
    }
  },
  computed:{
    getLevelOneData(){
      let pathes1=[
        {
          name:'skin',
          hanzi:'皮肤'
        },
        {
          name:'general',
          hanzi:'常见病症'
        }
      ]
      let pathes2=svgPath[this.svg.sex][this.svg.deriction].levelOne
      let pathes=pathes1.concat(pathes2)
      return pathes;
    },
    getLevelTwoData(){
      let levelTwo = svgPath[this.svg.sex][this.svg.deriction].levelTwo
      let _self=this
      let levelTwoData=[]
      levelTwoData = levelTwo.filter(function(item){
          return item.parentName==_self.currentLevelOneValue.name;
      })
      return levelTwoData;
    },
  }
};
</script>
<style>
.body-image {
  background: url("../assets/svgBg.png");
  background-size: cover;
  width:100vw;
  height:100vh;
}
.body-image .contro{
  width:10vw;
  position:fixed;
  right:0;
  top:0vh;
  display:flex;
  justify-content: center;
  flex-wrap: wrap;
  align-items: flex-start;
  z-index: 100000;
}
.body-image .contro span{
  display: inline-block;
  border:1px solid #FFFFFF;
  display:inline-flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  padding:2vh 0;
}
.body-image .contro span font{
  color:#FFFFFF;
  font-size: 0.5em;
  text-align: center;
}
.body-image .contro span i,font{
  color:#FFFFFF;
  font-size: 1em;
}
.van-popup{
  overflow-y: scroll;
  text-align: center;
}
#full-body {
  display: block;
  width:100%;
  height:100%;
}
#full-body .layers {
  cursor: pointer;
  display: none;
  opacity: 0;
  outline: 0;
  transition: opacity 0.5s linear;
}
#full-body .layers:hover {
  opacity: 0.525;
  outline: 0;
  transition: opacity 0.5s linear;
}
#full-body .layer-show {
  display: block;
}
</style>
