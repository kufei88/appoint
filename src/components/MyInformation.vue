<template>
  <div style="padding-bottom: 50px">
    <x-header :left-options="{showBack: false}" class="header">个人中心</x-header>
    <div class="main">
      <grid :cols="5" class="header" :show-vertical-dividers="false">
        <grid-item>
          <img :src="headImgUrl" class="headImg" />
        </grid-item>
        <grid-item style="width:200px">
          <div class="nick">卡号:{{number}}</div>
          <div class="rank">IC卡余额:{{money.toFixed(2)}}</div>
        </grid-item>
      </grid>

      <group :gutter="0" style="margin-top:300px">
        <x-button @click.native="exit">退出登录</x-button>
      </group>
    </div>
  </div>
</template>
<script>
import {
  XHeader,
  Divider,
  XInput,
  Group,
  XButton,
  Toast,
  Cell,
  Qrcode,
  Flexbox,
  FlexboxItem,
  CellBox,
  Grid,
  GridItem
} from "vux";
import HttpUtil from "../util/HttpUtil";
import MyUtil from "../util/MyUtil";
import { mapState } from "vuex";
export default {
  data() {
    return {
      number: "",
      code: "",
      money: 0,

      givingMoney: 0,
      rebatesMoney: 0,
      rechargeRebates: 0,
      resultsRebates: 0,
      memberType: "",
      shopcode:
        "https://open.weixin.qq.com/connect/oauth2/authorize?appid=wxc8423586aa004b0a&redirect_uri=http%3a%2f%2fwww.jishengsoft.com%2fredirect%2fredirectwemall.asp&response_type=code&scope=snsapi_userinfo&state=STATE#wechat_redirect"
    };
  },
  mounted() {
    this.getInformation();
  },
  methods: {
    getInformation() {
      let _this = this;
      console.log(this.$store.state.vux.snsUserInfo.username);
      HttpUtil.get(
        "wechat/getClientInfo.do",
        { client: this.$store.state.vux.snsUserInfo.username },
        function(data) {
          _this.money = data.balance;
          _this.number = data.card;
        }
      );
    },
    exit() {
      let _this = this;

      HttpUtil.get(
        "wechat/delWxid.do",
        { client: this.$store.state.vux.snsUserInfo.openId },
        function(data) {
          _this.$store.commit("updateUsername", "");
          _this.$router.push({ path: "/" });
        }
      );
    }
  },
  computed: {
    ...mapState({
      nick: state => state.vux.snsUserInfo.nickname,
      headImgUrl: state => state.vux.snsUserInfo.headImgUrl,
      username: state => state.vux.snsUserInfo.username
    })
  },

  components: {
    XHeader,
    Divider,
    XInput,
    Group,
    XButton,
    Toast,
    Cell,
    Qrcode,
    Flexbox,
    FlexboxItem,
    CellBox,
    Grid,
    GridItem
  }
};
</script>
<style scoped>
.header {
  background: linear-gradient(90deg, #1a961a, #6bc2f7);
  box-shadow: 0 2px 5px rgba(255, 98, 98, 0.4);
}
.headImg {
  width: 50px;
  height: 50px;
  float: left;
}
.nick {
  font-size: 16px;
  color: #fff;
}
.rank {
  font-size: 16px;
  color: #fff;

  border-radius: 5px;
}

.numberform h1 {
  text-align: center;
  color: #a1a1a1;
}
.moneyText {
  font-size: 9px;
  color: #000;
  padding: 10px 5px;
  background: linear-gradient(90deg, #ecf0fa, #f6f8fc);
}

.moneyBox {
  border-radius: 5px;
}
</style>
