
<template >
  <div v-if="unRegister">
    <x-header :left-options="{showBack: false}" class="header">经营户登录</x-header>
    <group title="请使用ic卡卡号和密码登录">
      <x-input title="卡号" placeholder="请输入卡号" v-model="card" required></x-input>
      <x-input title="密码" type="password" placeholder="请输入密码" v-model="password" required></x-input>
    </group>
    <div style="margin-top:50px">
      <x-button type="primary" @click.native="addClient">登录</x-button>
    </div>
  </div>
  <div v-else>
    <x-header :left-options="{showBack: false}" class="header">
      车辆预约信息
      <x-icon
        type="android-add-circle"
        size="30"
        @click="addCar"
        slot="right"
        style="margin-top:-5px"
      ></x-icon>
    </x-header>
    <tab v-model="filterType">
      <tab-item selected @on-item-click="filterList">预约</tab-item>
      <tab-item @on-item-click="filterList">进场</tab-item>
    </tab>
    <swipeout>
      <swipeout-item transition-mode="follow" v-for="item in appointList" :key="item.id">
        <div slot="right-menu">
          <swipeout-button @click.native="delAppoint(item)" type="warn">删除</swipeout-button>
        </div>
        <div class="box" slot="content" @click="showAppoint(item)">
          <div class="car">{{item.plateNumber}}</div>
          <flexbox>
            <flexbox-item>
              <div class="owner">
                {{item.goodsType}}
                <span>|</span>
                {{item.ton}}吨
              </div>
            </flexbox-item>
            <flexbox-item>
              <div class="changeType">{{item.changeType}}</div>
            </flexbox-item>
          </flexbox>
          <flexbox>
            <flexbox-item>
              <div class="address">{{item.address}}</div>
            </flexbox-item>
            <flexbox-item>
              <div class="changeType">{{item.appointTime}}</div>
            </flexbox-item>
          </flexbox>
        </div>
      </swipeout-item>
    </swipeout>
    <!--
    <div class="round-click" @click="addCar">
      <a>+</a>
    </div>
    -->
  </div>
</template>

<script>
import {
  Tabbar,
  TabbarItem,
  Group,
  Cell,
  XHeader,
  CellBox,
  Panel,
  AjaxPlugin,
  TransferDom,
  Previewer,
  XButton,
  XInput,
  XTextarea,
  XDialog,
  FormPreview,
  Flexbox,
  FlexboxItem,
  Msg,
  Tab,
  TabItem,
  GroupTitle,
  Datetime,
  Selector,
  Grid,
  GridItem,
  Swipeout,
  SwipeoutItem,
  SwipeoutButton,
  Icon
} from "vux";
import GoodsList from "./GoodsList";
import HttpUtil from "../util/HttpUtil";
import MyUtil from "../util/MyUtil";

export default {
  directives: {
    TransferDom
  },
  data() {
    let data = [];

    return {
      unRegister: true,
      birthday: "1990-01-01",
      sexList: ["男", "女"],
      sex: "男",
      card: "",
      password: "",
      client: "",
      tel: "",
      headImgUrl: "",
      couponsCount: 0,
      money: 0,
      integral: 0,
      goodsType: [],
      appointList: [],
      appointAllList: [],
      typeFilter: 0
    };
  },
  mounted() {
    let _this = this;
    console.log(this.$store.state.vux.snsUserInfo);
    if (!this.$store.state.vux.snsUserInfo.username) {
      if (this.$store.state.vux.snsUserInfo.openId) {
        _this.unRegister = true;
        _this.wxid = _this.$store.state.vux.snsUserInfo.openId;
        return;
      }
      let code = _this.$geturlpara.getUrlKey("code");
      let state = _this.$geturlpara.getUrlKey("state");
      let username = _this.$geturlpara.getUrlKey("username");
      this.$store.commit("updateTenant", { tenant: username });
      HttpUtil.get(
        "wechat/clientLogin.do",
        { code: code, state: state },
        function(data) {
          _this.wxid = data.openId;
          _this.headImgUrl = data.headImgUrl;
          _this.$store.commit("updateSnsUserInfo", data);
          console.log(JSON.stringify(data));
          _this.sex = data.sex == 1 ? "男" : "女";
          if (data.username != "null") {
            console.log(111111);
            _this.unRegister = false;
            _this.$store.commit("updateUsername", data.username);

            _this.client = data.username;
            _this.getApponitList();
          }
        }
      );

      /*
      HttpUtil.get('wechat/getWechatUser',{},function(data){

      })
      */
    } else {
      if (_this.$store.state.vux.snsUserInfo.username != "null") {
        _this.unRegister = false;
        _this.wxid = _this.$store.state.vux.snsUserInfo.openId;

        _this.client = _this.$store.state.vux.snsUserInfo.username;
        _this.number = _this.$store.state.vux.snsUserInfo.tel;
        _this.headImgUrl = _this.$store.state.vux.snsUserInfo.headImgUrl;
        _this.money = _this.$store.state.vux.snsUserInfo.advanceGathering;
        _this.integral = _this.$store.state.vux.snsUserInfo.integral;
        _this.getApponitList();
      }
    }
  },
  methods: {
    filterList(index) {
      let _this = this;
      console.log(this.typeFilter);
      this.appointList = this.appointAllList.filter(function(item) {
        return item.type == (index == 0 ? "预约" : "进场");
      });
    },
    showLoading() {
      this.$vux.loading.show({
        text: "正在登录"
      });
    },
    hideLoading() {
      this.$vux.loading.hide();
    },
    getApponitList() {
      let _this = this;
      HttpUtil.get(
        "wechat/getAppointList.do",
        { client: this.client },
        function(data) {
          _this.appointAllList = data;
          _this.filterList(0);
        }
      );
    },
    addCar() {
      this.$router.push({ path: "/addCar" });
    },
    delAppoint(item) {
      if (item.type == "进场") {
        return;
      }
      let _this = this;

      HttpUtil.get(
        "wechat/delAppoint.do",
        { billCode: item.billCode },
        function(data) {
          _this.getApponitList();
        }
      );
    },
    showAppoint(item) {
      if (item.type == "进场") {
        return;
      }
      console.log(item.billCode);
      this.$router.push({
        name: "addCar",
        params: { billCode: item.billCode }
      });
    },
    addClient() {
      if (!this.card) {
        this.$vux.alert.show({
          title: "提示",
          content: "请输入卡号"
        });
        return;
      }

      if (!this.wxid) {
        this.$vux.alert.show({
          title: "提示",
          content: "页面状态不对，请关闭重新进入"
        });
        return;
      }
      let _this = this;
      console.log(this.birthday);
      this.showLoading();
      HttpUtil.post(
        "wechat/login.do",
        {
          username: this.card,
          password: this.password,
          openid: this.wxid
        },
        function(data) {
          console.log(data);
          _this.hideLoading();
          if (data.result == "success") {
            _this.$store.commit("updateUsername", data.result1);
            _this.$store.commit("updateTenant", data.result2);
            _this.client = data.result1;

            _this.unRegister = false;
            _this.getApponitList();
          } else {
            _this.$vux.toast.show({
              text: "卡号或密码错误",
              type: "warn"
            });
          }
        },
        function(error) {
          _this.hideLoading();
        }
      );
    }
  },
  components: {
    Tabbar,
    TabbarItem,
    Group,
    Cell,
    XHeader,
    CellBox,
    Panel,
    GoodsList,
    Previewer,
    XButton,
    XInput,
    XTextarea,
    XDialog,
    FormPreview,
    Flexbox,
    FlexboxItem,
    Msg,
    Tab,
    TabItem,
    GroupTitle,
    Datetime,
    Selector,
    Grid,
    GridItem,
    Selector,
    Swipeout,
    SwipeoutItem,
    SwipeoutButton,
    Icon
  }
};
</script>
<style lang="scss" scoped>
.dr-user-info {
  width: 100%;
  height: 6.7667rem;
  position: relative;
  text-align: center;
  color: #000;
  background: url(../assets/img/user_bg.jpg) no-repeat top center;
  background-size: 100%;
}
.dr-user-info .avatar {
  width: 2.9733rem;
  height: 2.9733rem;
  border-radius: 50%;
  padding: 0.3867rem;
  overflow: hidden;
  margin: 0.5467rem 0.0933rem;
  background: #e7e7e7;
}
.dr-user-info .avatar .avatar-wrap {
  background: #a9cfa9;
  width: 2.4rem;
  height: 2.4rem;
  padding: 0.18rem;
  border-radius: 50%;
}
.dr-user-info .avatar .avatar-wrap img {
  width: 2.3733rem;
  height: 2.3733rem;
  border-radius: 50%;
  border: 1px solid #1a961a;
}
.dr-user-info .username {
  color: #689524;
  font-size: 16px;
}
.dr-user-info .userid {
  font-size: 0.6467rem;
  line-height: 1;
  margin-top: 0.5rem;
  position: relative;
  padding-right: 1.3133rem;
  display: inline-block;
  bottom: -1rem;
}
.logo {
  width: 150px;

  margin: 10px auto 0;
}
.vip {
  position: absolute;
  right: 10px;
  bottom: 20px;
  color: red;
}

.box {
  border-bottom: 1px solid #ececec;
  padding: 10px 10px 9px;

  background-color: #fff;
  position: relative;
}
.car {
  font-size: 14px;
  line-height: 15px;
  color: #222;
  height: 15px;
  overflow: hidden;
}
.owner {
  font-size: 12px;
  line-height: 17px;
  height: 17px;
  color: #999;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  margin: 3px 0 4px;
}
.owner span {
  font-size: 8px;
  padding: 0 5px;
  position: relative;
  top: -2px;
  color: #e1e1e1;
}
.changeType {
  font-size: 12px;
  color: #999;
  text-align: right;
  padding-right: 20px;
}
.address {
  font-size: 12px;
  line-height: 17px;
  height: 17px;
  color: #999;

  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  margin: 3px 0 4px;
}
.deal {
  position: absolute;
  right: 0;
  top: 15px;
}

.round-click {
  height: 40px;
  width: 40px;
  background-color: #1aad19;
  opacity: 0.7;
  padding: 0;
  position: fixed;
  bottom: 75px;
  right: 20px;
  border-radius: 100%;
  line-height: 40px;
  text-align: center;
  z-index: 90;
}
.round-click a {
  font-size: 42px;
  max-width: 40px;
  color: #fff;
}
.header {
  background-color: #1aad19;
}
</style>
