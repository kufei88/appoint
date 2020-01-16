<template>
  <div class="order" style="padding-bottom: 50px">
    <x-header :left-options="{showBack: true}">订单管理</x-header>
    <group title="订单信息">
      <bill-list :title="title"  :list="billlist" :showMenu="false" 　@on-del-item="delBill($event)" @on-edt-item="edtBill($event)" ></bill-list>


    </group>

  </div>
</template>

<script>
import { XHeader, Group, Confirm } from "vux";
import BillList from "./BillList.vue";
import HttpUtil from "../util/HttpUtil";
export default {
  name: "order",
  components: {
    XHeader,
    BillList,
    Group,
    Confirm
  },
  data() {
    return {
      billlist: [],
      title: "订单日期"
    };
  },
  mounted() {
    this.getBillList();
  },
  methods: {
    getBillList() {
      let _this = this;
      let templist = [];
      let billcode = "";
      let goodlist = [];
      let billdate = "";
      let audite = "";
      HttpUtil.get(
        "bill/getOrderByClient.do",
        { client: this.$store.state.vux.snsUserInfo.username },
        function(data) {
          data.forEach(function(item) {
            if (billcode != "" && billcode != item.goodCode) {
              templist.push({
                billdate: billdate,
                billcode: billcode,
                goodslist: goodlist,
                audite: audite
              });
              goodlist = [];
            }
            billdate = item.datetime;
            billcode = item.goodCode;
            audite = item.postition;

            goodlist.push({
              goodsName: item.stockGoodsName,
              number: item.stockGoodsNum
            });
          });
          templist.push({
            billdate: billdate,
            billcode: billcode,
            goodslist: goodlist,
            audite: audite
          });
          _this.billlist = templist;
        }
      );
    },
    delBill(e) {
      let _this = this;
      this.$vux.confirm.show({
        title: "提示",
        content: "是否确认删除该订单",

        onConfirm() {
          HttpUtil.del("bill/delOrder.do", { billCode: e }, function(data) {
            _this.getBillList();
          });
        }
      });
    },
    edtBill(e) {
      this.$router.push({ path: "/", query: { billcode: e } });
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
