<template>
  <div class="register">
    <p class="p_login">快捷登录</p>
    <div class="verify">
      <p class="bor_box">
        <!-- <img src="../../assets/per.png"> -->
        <cube-input
          v-model="phone"
          :placeholder="phoneplaceholder"
          :type="phonenumber"
          :maxlength="maxlength"
        ></cube-input>
      </p>
      <p class="verification bor_box">
        <!-- <img src="../../assets/passw.png"> -->
        <cube-input v-model="Sms" placeholder="请输入验证码" type="text" :maxlength="Smsmaxlength"></cube-input>
        <span @click="getVerification">{{verificationText}}</span>
      </p>
      <p class="forget">
        <span>未注册用户自动注册</span>
      </p>
    </div>
    <div class="protocol">
      <p class="check" @click="isImg">
        <img v-if="imgFalg" src="../../assets/face.png" />
      </p>我已阅读并同意
      <span>
        <router-link :to="{ path: '/agreement' }">《用户服务协议》</router-link>
      </span>
    </div>
    <p class="btn" @click="immediately">登录</p>
  </div>
</template>

<script>
import { codeLogin, getSms, forgetPwdAndRegister } from "../../api/app.api";
export default {
  name: "register",
  components: {},
  data() {
    return {
      appCode: this.common.wholesituation.appCode,
      title: "登录注册",
      phone: "",
      Sms: "", //短信验证码
      imgFalg: true,
      timestamp: Date.parse(new Date()), //获取时间
      text: "",
      time: null, //计时器
      verificationText: "获取验证码",
      verificationTrue: false, // 验证码按钮是否可点击
      intrada: false,
      id: "",
      phonenumber: "text",
      phoneplaceholder: "请输入手机号码",
      maxlength: 11,
      Smsmaxlength: 6,
      isCode: true, //验证码是否可点击状态
      strData: {},
      parameterData: {}
    };
  },
  methods: {
    isImg: function() {
    },
    Totas: function() {
      this.toast = this.$createToast({
        txt: this.text,
        type: "txt",
        time: 1000,
        onTimeout: () => {
          if (this.intrada) {
            this.$router.push({ name: "setPassword" });
          }
        }
      });
      this.toast.show();
    },
    immediately: function() {
      var pasVal = "^(?![0-9]+$)(?![a-zA-Z]+$)[0-9A-Za-z]{6,20}$";
      var re = new RegExp(pasVal);
      if (!this.verificationTrue) {
        this.text = "请输入正确的手机号！";
        this.Totas();
        return;
      } else if (!this.Sms) {
        this.text = "请输入短信验证码";
        this.Totas();
        return;
      }
      this.parameterData = {
        phone: this.phone,
        code: this.Sms,
        password: "",
        type: "1"
      };
      this.registerLogin();
    },
    //注册登录
    registerLogin: function() {
      codeLogin({
        phone: this.phone,
        validCode: this.Sms,
        deviceModel: this.strData.eCode,
        deviceUuid:this.strData.deviceCode,
        longitude: this.strData.longitude,//经度
        latitude: this.strData.latitude,//维度
        province: this.strData.province,//省份
        city: this.strData.city,//市
        county: this.strData.country,//国家
        region: this.strData.district,//区域
        street: this.strData.street,//街道
        address: this.strData.addrStr,//详细地址
        isA: this.strData.isA,//是否为代理 1 是 2 否
        systemLan: this.strData.systemLan,//系统语言
        sysVersion: this.strData.sysVersion,//系统版本
        appName: this.strData.appName,//APP名称
        appVersion: this.strData.appVersion,//APP版本
      })
        .then(res => {
          if (res.code == "200") {
            var resData = res.data;
            var resToken = res.data.loginToken;
            this.utils.save("token", resToken);
            this.utils.save("userInfo", resData);
            this.$router.push({ name: "home" });
          }
          if (res.code == "407") {
            this.text = "验证码错误";
            this.Totas();
            return;
          }
          this.text = res.msg;
          this.Totas();
        })
        .catch(error => {
          console.log(error);
        });
    },
    //获取验证码
    getVerification: function() {
      var str = dsBridge.call("loginSyn", "loginSyn");
      if (str != undefined) {
        this.strData = JSON.parse(str.replace("/", ""));
      }
      if (!this.verificationTrue) {
        this.text = "请输入正确的手机号！";
        this.Totas();
        return;
      }
      if (!this.isCode) {
        return;
      }
      this.countTime();
      getSms({
        phone: this.phone
      })
      .then(res => {
        this.text = res.msg;
        this.Totas();
      })
      .catch(error => {

      });
    },
    //校验电话号码，决定是否可点击获取验证码
    verificationPhone: function() {
      var reg = /^1[0-9]{10}$/;
      if (reg.test(this.phone)) {
        this.verificationTrue = true;
      } else {
        this.verificationTrue = false;
      }
    },
    //验证码的倒计时
    countTime: function() {
      let i = 59;
      this.isCode = false;
      this.verificationText = "59s";
      this.time = setInterval(() => {
        i--;
        this.verificationText = i + "s";
        if (i == 0) {
          this.verificationText = "获取验证码";
          this.Sms = "";
          this.verificationPhone();
          this.isCode = true;
          clearInterval(this.time);
        }
      }, 1000);
    },
  },
  mounted() {
    this.id = this.$route.query.id;
    
  },
  watch: {
    phone: function(v) {
      this.verificationPhone();
    }
  }
};
</script>

<style scoped lang="scss">
.register {
  width: 100%;
  background: #fff;
  border-radius: (30/75)+rem;
  padding: (90/75)+rem (30/75)+rem;
}
.p_login{
  color: #404040 ;
  font-size: (36/75)+rem;
  text-align: center;
}
img {
  display: block;
  width: 100%;
}
.main {
  padding: 0.8rem 0.5rem 0 0.5rem;
  font-size: 0.2rem;
}
.topLogo {
  width: 3.5rem;
  height: 3.5rem;
  margin: 0 auto;
}
.verify {
  padding: 0 (20/75)+rem;
  img{
    width: (42/75)+rem;
    height: (42/75)+rem;
    margin-right:(10/75)+rem;
  }
  .bor_box{
    border: 1px solid #ededed;
    display: flex;
    justify-content: flex-start;
    border-radius: (45/75)+rem;
    align-items: center;
    padding-left: (20/75)+rem;
  }
  margin-top: 1rem;
  .cube-input {
    position: relative;
    width: 100%;
    background: none;
    border-bottom: 0;
  }
  .graph {
    margin: 0.2rem 0;
    display: flex;
    justify-content: space-between;
    img {
      width: 0.9rem;
    }
  }
  .verification {
    position: relative;
    margin-top: 0.4rem;
    span {
      position: absolute;
      top: (5/75) + rem;
      right: -(10/75) + rem;
      display: block;
      width: 3rem;
      font-size: 0.35rem;
      color: #2B7EFD ;
      line-height: 1.2rem;
      text-align: center;
    }
  }
}
.pasword {
  margin-top: 0.2rem;
}
.forget {
  display: flex;
  justify-content: space-between;
  color: #a2b2c6;
  font-size: 0.35rem;
  padding-top: 0.5rem;
}
.check {
  width: 0.3rem;
  height: 0.3rem;
  border: 1px solid #2B7EFD;
  background: #2B7EFD;
  margin-right: 0.1rem;
}
.protocol {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  margin-top: 1.2rem;
  font-size: 0.35rem;
  color:  #969696;
  span {
    color:  #969696;
    a {
      color: #2B7EFD;
    }
  }
}
.btn {
  line-height: 1.2rem;
  background: #2B7EFD;
  font-size: 0.4rem;
  color: #fff;
  text-align: center;
  margin-top: 0.5rem;
  border-radius: (42/75)+rem;
}
.cube-input {
  font-size: 16px;
  line-height: 1.8;
  border-bottom: 1px solid #ededed;
}
</style>
