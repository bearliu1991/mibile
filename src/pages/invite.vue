<template>
    <div id="invite">
      <div v-if="isInvite && isRes" class="invite_wrap">
        <div class="text_cont">
            <h5 class="hi_name">Hi, &nbsp;<h3>{{userInfo.invitorName}}&nbsp;</h3>正在邀请您的加入</h5>
            <h3 class="corp_name">{{userInfo.corpName}}</h3>
            <h5 class="apart">
              {{userInfo.deptName}}<span v-if="userInfo.postName || userInfo.seatNo">({{userInfo.postName}}
                <span class="point_dot" v-if="userInfo.postName && userInfo.seatNo">&nbsp;.&nbsp;</span>
                <i v-if="userInfo.seatNo">坐席{{userInfo.seatNo}}</i>)</span>
            </h5>
        </div>
        <div class="process_p">
            <div class="wechat_info">
                <img v-if='userInfo.picUrl' :src="userInfo.picUrl">
                <img v-else src="@/assets/imgs/portrait.png">
                <div class="user">
                    <p>{{userInfo.nickname}}</p>
                    <span>{{userInfo.mobile}}</span>
                </div>
            </div>
            <div class="verify">
                <mt-field label="" v-model="captcha" placeholder="请输入验证码">
                    <mt-button type="primary" v-if="!isCount" @click="getCode">获取验证码</mt-button>
                    <mt-button disabled v-else>{{countNum}}秒后获取</mt-button>
                </mt-field>
            </div>
            <mt-button type="primary" size="large" class="join_btn" :class="{vaild: !!captcha}" @click="join">立即加入</mt-button>
            <p class="tips">若您的手机号码有误, 请联系邀请人修改</p>
        </div>
      </div>
      <Info v-if="!isInvite && isRes" :isSuccess="isSuccess" :userInfo="userInfo" :code="code">
      </Info>
    </div>
</template>
<script>
  import Info from './info'

  export default {
      name: 'info',
      data() {
          return {
            isInvite: true,
            userInfo: {},
            captcha: '',
            code: null,
            isCount: false,
            countNum: 60,
            timer: null,
            inviteCode: null,
            isSuccess: false,
            isRes: false,
            accountPlatformId: null,
            accountId: null
          }
      },
      created() {
        this.isInvite && this.init()
      },
      components: { Info },
      methods: {
        init() {
          document.title = '邀请加入'
          const path = location.href
          const arr = path.split(/[?=&#]/)
          const idIdx = arr.findIndex(item => item === 'eeId')
          const codeIdx = arr.findIndex(item => item === 'inviteCode')
          const formIdx = arr.findIndex(item => item === 'accountPlatformId')
          const accountIdx = arr.findIndex(item => item === 'accountId')
          let obj
          if (idIdx > -1 && (codeIdx > -1)) {
            this.accountPlatformId = arr[formIdx + 1]
            this.accountId = arr[accountIdx + 1]
            obj = {
                eeId: arr[idIdx + 1],
                inviteCode: arr[codeIdx + 1],
                accountPlatformId: arr[formIdx + 1],
                accountId: arr[accountIdx + 1]
              }
            this.inviteCode = arr[codeIdx + 1]
            this.getUserInfo(obj).then((res) => {
              this.isRes = true
              const data = res.data
              if (data.code === 1) {
                this.userInfo = data.data
                if (this.userInfo.joinStatus === 1) {
                  this.isSuccess = true
                  this.isInvite = false
                  document.title = '加入成功'
                }
              } else if (data.code === 3548) {
                this.isInvite = false
                document.title = '已绑定'
              } else if (data.code === 3547) {
                this.isInvite = false
                document.title = '已失效'
              }
            })
          }
        },
        join() {
          this.joinCorp({
            eeId: this.userInfo.eeId,
            inviteCode: this.inviteCode,
            identifyType: 'MOBILE',
            identifyCode: this.userInfo.mobile,
            validateCode: this.captcha,
            validateCodeType: 1,
            validateCodeLength: 4,
            accountPlatformId: this.accountPlatformId,
            accountId: this.accountId
          }).then((res) => {
            const data = res.data
            if (data.code === 3547 || (data.code === 3548)) {
              this.code = data.code
              this.isInvite = false
              data.code === 3547 && (document.title = '已失效')
              data.code === 3548 && (document.title = '已绑定')
              return false
            }
            let message = ''
            switch (data.code) {
              case 1:
                document.title = '加入成功'
                this.isSuccess = true
                this.isInvite = false
                return true
              case 2501:
                message = '验证码错误'
                break
              default:
                message = '数据有误'
                break
            }
            this.$toast({
              message,
              duration: 3000
            })
            return true
          })
        },
        getVerifyCode() {
          this.verifyCode({
            userId: this.userInfo.eeId,
            identifyType: 'MOBILE',
            identifyCode: this.userInfo.mobile,
            validateCodeType: 1,
            validateCodeLength: 4,
            validateCodeBusiType: 'INVITE_EMPLOYEE'
          }).then((res) => {
            const data = res.data
            if (data.code !== 1) {
              this.$toast({
                message: '获取验证码失败',
                duration: 3000
              })
            }
          })
        },
        getCode() {
          if (!this.isCount) {
            this.isCount = true
            this.getVerifyCode()
            this.timer = setInterval(() => {
              if (this.countNum > 0) {
                this.countNum = this.countNum - 1
              } else {
                this.isCount = false
                this.countNum = 60
                clearInterval(this.timer)
              }
            }, 1000)
          }
          return false
        }
      }
  }
</script>
<style lang="stylus">
  @import "~@/assets/css/index"
  #invite
    flex: 1
    width:100%
    height:100%
    .invite_wrap
      .text_cont
        setColumn()
        .hi_name
          fn(15px)
          weightN()
          setStart()
          margin: 0
          color(#606366)
          h3
            max-width: 130px
            ellipsis()
            color(#303133)
            display:inline
        .corp_name
          fn(20px)
          ellipsis()
          margin: 13px 0
        .apart
          fn(15px)
          // setStart()
          .point_dot
            inline()
            mb(10px)
            line-height:1px
      .process_p
        .mint-button
          fn(16px)
        .mint-button--normal
          padding:0
        .wechat_info
          setStart()
          mt(75px)
          img
            width: 50px
            height: 50px
            mr(15px)
          .user
            setMiddle()
            flex-direction:column
            p
              mb(5px)
              width:100%
              alignL()
        .verify
          margin: 20px 0
          mb(33px)
          .mint-cell-value
            input
              fn(15)
          .mint-cell-wrapper
            background-image:none
          button
            fn(15px)
            width:114px
        .join_btn
          height: 41px
          bgColor(#b2d8ff)
          &.vaild
            bgColor(text_color)
        .tips
          mt(10px)
          color(#999)
          center()
          fn(12px)
</style>

