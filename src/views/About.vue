<template>
  <svg>
    <path></path>
  </svg>
</template>
<script>
import svgPath from '../../public/json/svgPath'
export default {
  data(){
    return{
      sexType:'male',
      maleFrontOne: [],
      maleFrontTwo: [],
      maleBackOne: [],
      maleBackTwo: []
    }
  },
  mounted(){
    this.
    fetch('svgPath').then(response=>response.json()).then(res=>{
      console.log(res)
    })
  }
}
</script>
<style lang="scss">

</style>
