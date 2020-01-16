<template>
  <div>
    <x-header :left-options="{showBack: false}" class="header">一卡通对账</x-header>

    <flexbox>
      <flexbox-item>
        <datetime class="dateRange" v-model="beginDate" title="开始时间" @on-change="getTableData"></datetime>
      </flexbox-item>
      <flexbox-item>
        <datetime class="dateRange" v-model="endDate" title="结束时间" @on-change="getTableData"></datetime>
      </flexbox-item>
    </flexbox>
    <!--
    <x-table
      :cell-bordered="false"
      :content-bordered="false"
      style="font-size:16px;background-color:#fff;"
    >
      <thead style="background-color: #F7F7F7">
        <tr>
          <th>日期</th>
          <th>业务</th>
          <th>金额</th>
          <th>结余金额</th>
        </tr>
      </thead>
      <tbody>
        <div v-for="(item,index) in tableData" :key="index">
          <tr>
            <td>{{item.billDate}}</td>
            <td>{{item.type}}</td>
            <td>{{item.money}}</td>
            <td>{{item.balance}}</td>
          </tr>
          <tr>
            <td colspan="4">{{item.plate}}</td>
          </tr>
        </div>
      </tbody>
    </x-table>
    -->
    <div style="font-size:16px;background-color:#fff;">
      <flexbox style="background-color: #F7F7F7 ;margin-left:10px">
        <flexbox-item>日期</flexbox-item>
        <flexbox-item>业务</flexbox-item>
        <flexbox-item>金额</flexbox-item>
        <flexbox-item>结余金额</flexbox-item>
      </flexbox>
      <div v-for="(item,index) in tableData" :key="index" style="border-bottom:1px solid #ddd">
        <flexbox style="height:30px;margin-left:10px">
          <flexbox-item>{{item.billDate}}</flexbox-item>
          <flexbox-item>{{item.type}}</flexbox-item>
          <flexbox-item>{{item.money}}</flexbox-item>
          <flexbox-item>{{item.balance}}</flexbox-item>
        </flexbox>
        <flexbox
          style="height:30px;color:#6D6D72;font-size:9px;margin-left:10px"
          v-show="item.plate"
        >
          <span style="margin-right:10px">{{item.plate}}</span>
          <span style="margin-right:10px">{{item.address}}</span>
          <span style="margin-right:10px">{{item.goodsName}}</span>
          <span style="margin-right:10px">进场:{{item.enterTime}}</span>

          <span style="margin-right:10px">{{item.remark}}</span>
        </flexbox>
      </div>
    </div>
  </div>
</template>

<script>
import HttpUtil from "../util/HttpUtil";
import { XNumber, XHeader, Flexbox, FlexboxItem, Datetime, XTable } from "vux";
import MyUtil from "../util/MyUtil";
export default {
  components: {
    XNumber,
    XHeader,
    Flexbox,
    FlexboxItem,
    Datetime,
    XTable
  },
  data() {
    return {
      beginDate: "",
      endDate: "",
      beginDate1: "",
      endDate1: "",
      tableData: []
    };
  },
  mounted() {
    let now = new Date();
    this.beginDate =
      now.getFullYear() + "-" + this.formatDay(now.getMonth() + 1) + "-01";
    console.log(this.beginDate);
    this.beginDate1 = this.beginDate;
    this.endDate =
      now.getFullYear() +
      "-" +
      this.formatDay(now.getMonth() + 1) +
      "-" +
      now.getDate();
    this.endDate1 = this.endDate;
    this.getTableData();
  },
  methods: {
    formatDay(day) {
      // console.log(day)
      // console.log(day < 10)
      return day < 10 ? "0" + day : day;
    },
    getTableData() {
      let _this = this;

      HttpUtil.get(
        "wechat/getCheckAccountList.do",
        {
          client: this.$store.state.vux.snsUserInfo.clientPrincipal,
          beginDate: this.beginDate,
          endDate: this.endDate
        },
        function(data) {
          _this.tableData = data;
          if (!_this.beginDate) {
            _this.beginDate = _this.beginDate1;
          }

          if (!_this.endDate) {
            _this.endDate = _this.endDate1;
          }
        }
      );
    },
    delBill(item) {
      this.$emit("on-del-item", item);
    },
    edtBill(src) {
      this.$emit("on-edt-item", src);
    },
    findbyindex(currentValue) {
      return (currentValue.index = this.index);
    }
  }
};
</script>
<style scoped>
.dateRange {
  font-size: 12px;
}
.clear {
  clear: both;
}
.header {
  background: linear-gradient(90deg, #1a961a, #6bc2f7);
  box-shadow: 0 2px 5px rgba(255, 98, 98, 0.4);
}
#emptylist {
  margin-bottom: 20px;
  padding-top: 60px;
  font-size: 14px;
  display: block;
  text-align: center;
  padding: 30px 10px;
  color: #999;
  width: 100%;
  padding: 20px 10px;
  vertical-align: middle;
  text-align: center;
  color: #999;
  font-size: 12px;
  line-height: 20px;
  -webkit-box-sizing: border-box;
  -moz-box-sizing: border-box;
  box-sizing: border-box;
}

#emptylist h4 {
  font-size: 16px;
  margin-bottom: 10px;
  color: #666;
}

.tag {
  color: #f60;
  border-color: #f60;
  font-size: 14px;
  line-height: 18px;
  display: inline-block;
  background-color: transparent;
  border: 1px solid #f60;
  border-radius: 3px;
  text-align: center;
  margin: 0;
  text-decoration: none;
}

#billlist {
  background: #f0f2f5;
  padding-top: 5px;
}

.ubilllist {
  list-style: none;
  margin: 0;
  padding: 0;
  font-size: 0;
}

.bill {
  background: #fff;
  margin-top: 5px;
  border-bottom: 1px solid #ddd;
  border-top: 1px solid #ddd;
  position: relative;
}

.title p {
  margin: 0 10px;
  line-height: 40px;
  height: 40px;
  font-size: 15px;
  float: left;
}

.title a {
  font: 18px/30px "微软雅黑";
  float: right;
  text-decoration: none;
  color: red;
  border: 1px solid red;
  width: 50px;
  text-align: center;
  border-radius: 5px;
  height: 30px;
  margin: 5px;
}

.title {
  height: 40px;
  border-bottom: 1px solid #ddd;
}

.goodslist {
  font-size: 0;
  padding: 0 15px;
}

.goodslist li {
  display: inline-block;
  font-size: 12px;
  width: 100%;
  height: 1.2rem;
  line-height: 1.2rem;
}

.signet {
  z-index: 2;

  right: 30px;
  top: 20%;
  width: 64px;
  height: 64px;
  background: url(../assets/index.png) no-repeat;
  background-size: 64px 64px;
}

.item-pay-data {
  float: right;
}

.price {
  font-weight: 700;
  text-overflow: ellipsis;
}

.num {
  font-size: 12px;
  color: #999;
}

.sum {
  float: right;
}

.sum b {
  word-break: break-all;
  font-weight: 700;
}
.goods {
  font-weight: 400;
  font-size: 12px;
}
</style>

