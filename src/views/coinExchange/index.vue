<template>
  <div class="pageWrap bg_wrap dis-table">
    <explorer></explorer>

    <div class="page-main content-middle">
      <div class="p-title">
          <div class="logo"><img src="../../assets/img/logo_coinExchange.png"/></div>
          <span>欢迎来到TDOS资产互换</span>
      </div>

      <div class="page-content">
        <div class="searchInBox border-box">
           <input type="text" placeholder="您可在此输入复制的哈希值以此查询" v-model = "search_hash"/>
           <a class="btn-search pointer" @click="showResult">

           </a>
        </div>
        <div class="coin-box" :class="{'coin-box-change':isExchange}">
          <div class="coin-col">
            <div class="f-col">
              <div class="coin_name">
                <div class="fr">{{coin1.nick}}</div>
                <span class="s1">{{coin1.name}}</span>
              </div>
              <div class="din-box">
                <div class="din"><input type="text" :placeholder="coin1.nick" v-limitNum ref="amount" /></div>
              </div>
            </div>
            <p class="mess">{{coin1.proportion}}</p>
          </div>
          <a class="icon_exchange pointer" @click="coinExchange"></a>
          <div class="coin-col">
            <div class="f-col">
              <div class="coin_name">
                <div class="fr">{{coin2.nick}}</div>
                <span class="s2">{{coin2.name}}</span>
              </div>
              <div class="din-box">
                <div class="din">
                  <p>{{coin2.nick}}</p>
                </div>
              </div>
            </div>
            <p class="mess">{{coin2.proportion}}</p>
          </div>
        </div>

        <div class="btnbox">
          <a class="pointer btn-exchange" @click="confirmExchange">确认转换</a>
          <a ref="sendTx"></a>
        </div>

      </div>


    </div>

    <!--转换成功展示-->
    <div class="trans" v-if="exchangeSuc">
      <div class="dis-table">
        <div class="popup-div">
          <div class="popup-main">
            <div class="popup-header">
              代币转换成功
              <a class="pointer close" @click="exchangeSuc=false;addScroll()"></a>
            </div>
            <div class="popup-content chain-content">
              <p class="chain-title">恭喜您！转换成功！该笔交易已存证上链！以下是您的上链哈希：</p>
              <div class="hash-value">
                <span>{{hash}}</span>
                <a class="copy pointer" v-clipboard:copy="message" v-clipboard:success="onCopy" v-clipboard:error="onError"></a>
              </div>
              <p class="p-mess">（您可复制上方哈希值并点击右上角按钮至“TDS浏览器”查询。）</p>
            </div>
            <div class="btnbox">
              <a class="cf-btn pointer bot-btn" @click="exchangeSuc=false;addScroll()">我知道了</a>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!--查询结果-->
    <div class="trans" v-if="searchResult">
      <div class="dis-table">
        <div class="popup-div dis-table">
          <div class="popup-main result-popup-main">
            <div class="popup-header">
              查询信息
              <a class="pointer close" @click="searchResult=false;addScroll()"></a>
            </div>
            <div class="popup-content result-content">
              <div class="result-col">
                <span class="lab">互换信息</span>
                <p>事务hash：{{hash}}</p>
                <p><span>{{from}}:{{searchAmount}}</span><img class="arrow" src="../../assets/img/arrow.png"/><span>{{to}}:{{from == 'ETH'? 0.5 *searchAmount: 2 * searchAmount }}</span></p>
                <p>互换比率：{{rate}}</p>
              </div>

            </div>
            <div class="btnbox">
              <a class="cf-btn pointer bot-btn" @click="searchResult=false;addScroll()">我知道了</a>
            </div>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>


<script>
  import explorer from '@/components/browser1.vue'
  import TpScroll from '@/assets/js/tp-scroll.js'
  import { getETB, getBTE, saveChange,getChange,getTransaction } from '@/api/dapps'
  import {showLoading, hideLoading} from '@/assets/js/loading'

  export default {
    data () {
      return {
        type: 1,//头部右上角浏览器

        exchangeSuc: false,//转换成功展示

        coin1: {
          name: '以太坊', nick: 'ETH', proportion: ''
        },
        coin2: {
          name: '比特币', nick: 'BTC', proportion: ''
        },

        isExchange: false,//货币顺序是否转换
        // amount:'',
        searchResult:false,//查询结果
        hash:'',
        rate:'',
        from:'',
        to:'',
        searchAmount: 0,
        hash:'',
        message:'',
        search_hash:''
      }
    },
    components: {
      explorer
    },
    filters: {
      ellipsis (value) {
        if (!value) return ''
        if (value.length > 74) {
          return value.slice(0, 74) + '...'
        }
        return value
      }
    },
    methods: {

      //点击货币中间转换按钮
      coinExchange () {
        let that = this
        that.isExchange = !that.isExchange
        let obj1 = JSON.parse(JSON.stringify(that.coin2))
        let obj2 = JSON.parse(JSON.stringify(that.coin1))
        // that.amount = that.amount.replace(/[^\d]/g, "")
        that.coin1 = obj1
        that.coin2 = obj2
        // console.log(that.amount)
      },
      //点击查询
      showResult(){
        let that =  this;
        TpScroll.RemoveScroll()
        that.getCoinChange(that.search_hash);
      },
      //点击确认转换
      async confirmExchange () {
        let that = this
        let amount = this.$refs.amount.value
        if (amount == ""){
          return that.$toast("请输入数量", 3000);
        }
        if (amount > 10000000){
          return that.$toast("数量需要小于10,000,000", 3000);
        }
        TpScroll.RemoveScroll()
        let payload = {
          from: that.coin1.nick, to: that.coin2.nick, amount: amount, rate:that.coin1.proportion
        }

        let pk = that.getPK();
        if (pk == "") {
          return that.$toast("获取账户失败，请打开TDOS插件", 3000);
        }
        let Change = await saveChange(payload, pk)
        let sendTx = JSON.stringify(Change);
        that.$refs.sendTx.href =
          "javascript:sendMessageToContentScriptByPostMessage('" + sendTx + "')";
        that.$refs.sendTx.click();
        that.$refs.amount.value = ''
        return that.$toast("事务已生成，请打开TDOS插件进行广播", 3000);
      },

      addScroll () {
        TpScroll.AddScroll()
      },

      async get () {
        let that = this
        let Eth = await getETB();
        let Btc = await getBTE();
        that.coin2.proportion = "1BTC="+Eth+"Eth";
        that.coin1.proportion = "1ETH="+Btc+"Btc";
      },
      async getCoinChange(hash){
        let that = this
        let u;
        try {
          u = await getChange(hash);
        }catch(err){
          return that.$toast("事务hash不存在", 3000);
        }
        if (u == ""){
          return that.$toast("事务hash不存在", 3000);
        }
          that.searchResult = true;
          that.searchAmount = u.amount;
          that.from = u.from;
          that.to = u.to;
          that.rate = u.rate;
        return u;
        // that.searchAmount = u.amount;
        // that.from = u.from;
        // that.to = u.to;
        // that.rate = u.rate;
      },
      timer_tx () {
        let that = this
        let hash = that.getRes().trim()
        if (hash != '') {
          that.hash = hash;
          showLoading('事务广播成功，事务哈希为：\n' + hash+","+'\n' + '请等待上链...')
          this.timer1 = setInterval(function () {
            getTransaction(hash).then(tx => {
              if (tx.confirms != -1) {
                that.message = hash
                hideLoading()
                clearInterval(that.timer1)
                that.exchangeSuc = true;
              }
            })

          }, 1000)
        }
      },
      onCopy: function (e) {
        let that = this
        return that.$toast('复制成功', 2000)
      },
      onError: function (e) {
        let that = this
        return that.$toast('复制失败，请稍后重试', 2000)
      }

    },
    mounted: function () {
      this.get();
      this.timer = setInterval(this.timer_tx, 1000)
    },
    beforeDestroy() {
      clearInterval(this.timer)
      if (this.timer1) {
        clearInterval(this.timer1)
      }
    }
  }
</script>
<style scoped lang="less">
  @import url(../../assets/less/crowdFunding.less);
  @import url(../../assets/less/coinExchange.less);

</style>>
