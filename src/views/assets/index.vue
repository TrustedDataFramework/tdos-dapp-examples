<template>
  <div class="pageWrap f-wrap-table pw-pageWrap">
    <explorer :isHome="isHome" :type="type"></explorer>
      <div class="logo-intro">
         <div class="logo"><img src="../../assets/img/logo_assets.png"/></div>
         TDOS资产
      </div>
      <div class="page-main content-middle">

          <div class="btnBox pw_btnbox">
              <a class="pointer h-btn h-btn-1" @click="platform"><span>登记平台</span></a>
              <a class="pointer h-btn h-btn-2" @click="heir"><span>继承人</span></a>
          </div>
      </div>
  </div>
</template>



<style scoped lang="less">
@import url(../../assets/less/assets.less);
</style>>

<script>
import explorer from '@/components/browser1.vue'
export default {
    data(){
        return{
           isHome:true,
           type:1,
        }
    },
    components:{
     explorer
    },
    methods:{
      platform(){
        let that = this;
        that.$router.push({path:'/assets/onChain'})
      },
      heir(){
        let that = this;
        that.$router.push({path:'/assets/search'})
      }
    }
}
</script>
