<template>
  <div class="pageWrap bg-wrap" @click="isOnchain=false">
    <explorer></explorer>
    <div class="page-main">
      <div class="p-container">
        <h3 class="page-title">欢迎来到TDOS加密熊</h3>
        <div class="bear-content">
          <div class="bc-left">
            <div class="d-logo">
              <div class="box-shadow"></div>
              <div class="pic"></div>
            </div>
            <div class="btnbox">
              <!--等级升级到三级的时候按钮置灰不可再升级 添加class  c-btn-disable-->
              <a class="pointer cb-btn upbtn " :class="{'c-btn-disable':level==3}" @click="submit">升级</a>
              <a ref="sendTx"></a>
            </div>
          </div>
          <div class="bc-right">
            <a class="pointer mark" @click.stop="showInfo"></a>
            <div class="detailInfo">
              <h3 class="h-level">
                当前等级<span class="s-level"><a id="level"></a></span>
              </h3>
              <div class="info-col info-col1">
                <span class="lab">姓名：</span>
                <div class="d-info">
                  <span class="st">加密熊</span>
                </div>

              </div>
              <div class="info-col info-col2">
                <span class="lab">出生：</span>
                <div class="d-info">
                  <span class="st">1997-06-09</span>
                </div>
              </div>
              <div class="info-col info-col3">
                <span class="lab">攻击：</span>
                <div class="d-info">
                  <span class="st"><a id="aggressivity"></a></span>
                </div>
                <span class="upgrade"></span>
              </div>
              <div class="info-col info-col4">
                <span class="lab">血量：</span>
                <div class="d-info">
                  <span class="st"><a id="bloodvolume"></a></span>
                </div>
                <span class="upgrade"></span>
              </div>
              <div class="info-col info-col5">
                <span class="lab">身高：</span>
                <div class="d-info">
                  <span class="st"><a id="stature"></a></span>
                </div>
                <span class="upgrade"></span>
              </div>
              <div class="info-col info-col6">
                <span class="lab">吨位：</span>
                <div class="d-info">
                  <span class="st"><a id="tonnage"></a></span>
                </div>
                <span class="upgrade"></span>
              </div>
              <div class="info-col info-col7">
                <span class="lab">防御：</span>
                <div class="d-info">
                  <span class="st"><a id="defense"></a></span>
                </div>
                <span class="upgrade"></span>
              </div>
            </div>


            <!--等级上链信息-->
            <div class="chinInfo" v-show="isOnchain">
              <div>区块高度：{{height}}</div>
              <div class="p-line1">
                区块哈希：{{blockHash}}
                <a class="pointer a-copy" v-clipboard:copy="message2" v-clipboard:success="onCopy"
                   v-clipboard:error="onError"></a>
              </div>
              <div class="p-line1">
                事务哈希：{{hash}}
                <a class="pointer a-copy" v-clipboard:copy="message" v-clipboard:success="onCopy"
                   v-clipboard:error="onError"></a>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>


  </div>
</template>
<script>
  import explorer from '@/components/browser1.vue'
  import {hasBear, getBear, buyLevel, getTransaction} from '@/api/dapps'
  import {showLoading, hideLoading} from '@/assets/js/loading'

  export default {
    data () {
      return {
        isOnchain: false,
        hash: '',
        height: 0,
        blockHash: '',
        message: '',
        message2: '',
        level:0 //当前等级
      }
    },
    components: {
      explorer
    },

    methods: {

      onCopy: function (e) {
        let that = this
        return that.$toast('复制成功', 2000)
      },
      onError: function (e) {
        let that = this
        return that.$toast('复制失败，请稍后重试', 2000)
      },

      async showInfo () {
        let that = this
        that.isOnchain = !that.isOnchain

      },
      async get () {
        let that = this
        let pk = that.getPK()
        if (pk == '') {
          return that.$toast('获取账户失败，请打开TDOS插件', 3000)
        }
        let has = await hasBear(pk)
        if (!has) {
          that.$router.push({path: '/cryptoBear'})
        }
        let bear = await getBear(pk)
        that.hash = bear.hash
        that.message = bear.hash
        await getTransaction(that.hash).then(t => {
          that.height = t.blockHeight
          that.blockHash = t.blockHash
          that.message2 = t.blockHash
        })
        document.getElementById('aggressivity').innerHTML = bear.aggressivity
        document.getElementById('defense').innerHTML = bear.defense
        document.getElementById('stature').innerHTML = (bear.stature / 100) + 'M'
        document.getElementById('tonnage').innerHTML = bear.tonnage + 'T'
        document.getElementById('bloodvolume').innerHTML = bear.bloodvolume / 100
        document.getElementById('level').innerHTML = bear.level
        that.level = bear.level
      },
      async submit () {
        let that = this
        if(that.level==3){
          return;
        }
        let pk = that.getPK()
        if (pk == '') {
          return that.$toast('获取账户失败，请打开TDOS插件', 3000)
        }
        let has = await hasBear(pk)
        if (!has) {
          that.$toast('您还没有购买加密熊', 3000)
          return that.$router.push({path: '/cryptoBear'})
        }
     
        let Level = await buyLevel(pk)
        let sendTx = JSON.stringify(Level)
        that.$refs.sendTx.href =
          'javascript:sendMessageToContentScriptByPostMessage(\'' + sendTx + '\')'
        that.$refs.sendTx.click()
        return that.$toast('事务已生成，请打开TDOS插件进行广播', 3000)
      },
      async timer_tx () {
        let that = this
        let hash = that.getRes().trim()

        if (hash != '') {
          showLoading('事务广播成功，事务哈希为：\n' + hash + ',' + '\n' + '请等待上链...')
          this.timer1 = setInterval(function () {
            getTransaction(hash).then(tx => {
              if (tx.confirms != -1) {
                hideLoading()
                clearInterval(that.timer1)
                that.$router.push({path: '/cryptoBear'})
              }

            })
          }, 1000)
        }
      }
    },
    mounted () {
      this.get()
      this.timer = setInterval(this.timer_tx, 1000)
    },
    beforeDestroy () {
      clearInterval(this.timer)
      if (this.timer1) {
        clearInterval(this.timer1)
      }
    }
  }
</script>
<style scoped lang="less">
  @import url(../../assets/less/cryptoBear.less);
</style>
