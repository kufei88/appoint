<template>
  <div id="app">
    <!-- main content -->
    <!-- drawer content -->
    <view-box ref="viewBox">
      <loading v-model="isLoading"></loading>
      <router-view class="router-view"></router-view>
      <tabbar icon-class="vux-center" class="vux-demo-tabbar" slot="bottom" style="position:fixed">
        <tabbar-item :selected="$route.path === '/'" link="/">
          <span slot="icon" class="iconfont">&#xe614;</span>
          <span slot="label">车辆预约</span>
        </tabbar-item>
        <tabbar-item :selected="$route.path === '/SellList'" link="/SellList">
          <span slot="icon" class="iconfont">&#xe858;</span>
          <span slot="label">一卡通对账</span>
        </tabbar-item>

        <tabbar-item :selected="$route.path === '/my'" link="/my">
          <span slot="icon" class="iconfont">&#xe61e;</span>
          <span slot="label">我的</span>
        </tabbar-item>
      </tabbar>
    </view-box>
  </div>
</template>

<script>
import { Tabbar, TabbarItem, Group, Cell, Drawer, ViewBox, XHeader } from "vux";
import svgIcon from "./components/svg";
import { Loading } from "vux";
import { mapState } from "vuex";
export default {
  name: "app",
  mounted() {},
  components: {
    Tabbar,
    TabbarItem,
    Group,
    Cell,
    ViewBox,
    Drawer,
    XHeader,
    Loading,
    svgIcon
  },
  data() {
    return {
      drawerVisibility: false,
      showModeValue: "push",
      showPlacementValue: "left"
    };
  },
  computed: {
    ...mapState({
      isLoading: state => state.vux.isLoading
    }),
    leftOptions() {
      return {
        showBack: this.route.path !== "/"
      };
    }
  }
};
</script>

<style lang="less">
@import "~vux/src/styles/reset.less";
@import "./styles/font.less";

body {
  background-color: #fbf9fe;
}
.router-view {
  width: 100%;
  top: 46px;
}
.weui-tabbar {
  position: fixed;
}
</style>
