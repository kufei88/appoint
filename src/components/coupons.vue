<template>
  <div>
    <x-header :left-options="{showBack: true}" class="header">优惠券</x-header>
    <div class="coupon-list">
      <p class="title"></p>
      <div
        class="coupon-item flex"
        v-for="item in couponsList"
        v-bind:key="item.id"
        @click="couponClick(item)"
      >
        <div class="cell align-center fixed">
          <p class="type">{{item.couponName}}</p>
          <p class="limit">
            NO.{{item.couponNumber}}
            <br>
            有效期
            {{item.couponTime}}至{{item.endDate}}
          </p>
        </div>
        <div class="cell align-center value">
          {{item.couponMoney}}
          <span class="tag">元</span>
        </div>
      </div>
    </div>
    <div v-transfer-dom>
      <popup v-model="showQR" position="bottom" should-scroll-top-on-show style="text-align:center">
        <divider>劵号:{{couponNumber}}</divider>
        <barcode :value="couponNumber" :options="{ displayValue: false }"></barcode>
        <qrcode :value="couponNumber" type="img" :size="160"></qrcode>
      </popup>
    </div>
  </div>
</template>
<script>
import {
  Group,
  Cell,
  XHeader,
  CellBox,
  Popup,
  Divider,
  Qrcode,
  TransferDom
} from "vux";
import HttpUtil from "../util/HttpUtil";

export default {
  directives: {
    TransferDom
  },
  components: {
    Group,
    Cell,
    XHeader,
    CellBox,
    Popup,
    Divider,
    Qrcode
  },
  data() {
    return {
      couponsList: [],
      showQR: false,
      couponNumber: ""
    };
  },
  mounted() {
    this.getCouponsList();
  },
  methods: {
    getCouponsList() {
      let _this = this;
      HttpUtil.get(
        "wechat/getCouponsList.do",
        { client: this.$store.state.vux.snsUserInfo.username },
        function(data) {
          console.log(data);
          _this.couponsList = data;
        }
      );
    },
    couponClick(item) {
      this.couponNumber = item.couponNumber;
      this.showQR = true;
      console.log(item);
    }
  }
};
</script>
<style scoped>
.coupon-list {
  padding: 0 0.4rem;
}
.coupon-list .title {
  text-align: left;
  line-height: 1;
  margin-bottom: 0.36rem;
  font-size: 0.3467rem;
}
.coupon-list .coupon-item {
  width: 12.2rem;
  height: 4.6667rem;
  background: #fff url(../assets/img/yhq_bg1.jpg) no-repeat 0 0;
  background-size: cover;
  padding: 0.2667rem 0.5867rem 0 0.56rem;
}
.flex {
  display: -webkit-box !important;
  display: -webkit-flex !important;
  display: -ms-flexbox !important;
  display: flex !important;
  -webkit-flex-wrap: wrap;
  -ms-flex-wrap: wrap;
  flex-wrap: wrap;
}
.flex > .cell.fixed {
  -webkit-box-flex: 0 !important;
  -webkit-flex: none !important;
  -ms-flex: none !important;
  flex: none !important;
  width: auto;
}
.flex > .cell {
  -webkit-box-flex: 1;
  -webkit-flex: 1;
  -ms-flex: 1;
  flex: 1;
  width: 0;
  -webkit-flex-basis: 0;
  -ms-flex-preferred-size: 0;
  flex-basis: 0;
  max-width: 100%;
  display: block;
  padding: 0 !important;
  position: relative;
}
.coupon-list .coupon-item .type {
  font-size: 0.88rem;
  color: #689524;
  height: 2.0133rem;
  line-height: 2.0133rem;
}
.coupon-list .coupon-item .limit {
  font-size: 0.48rem;
  line-height: 0.6467rem;
  color: #8b8f96;
}
.coupon-list .coupon-item .value {
  font-size: 2.1467rem;
  color: #689524;
  line-height: 2.3467rem;
  text-align: right;
}
.flex > .cell.align-center {
  -webkit-align-self: center;
  -ms-flex-item-align: center;
  align-self: center;
}
.coupon-list .coupon-item .value .tag {
  font-size: 0.32rem;
  color: #8b8f96;
}
</style>
