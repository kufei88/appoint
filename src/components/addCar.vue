<template>
  <div>
    <x-header :left-options="{showBack: true}" class="header">新增预约</x-header>

    <group>
      <x-input
        title="车牌"
        v-model="plateNumber"
        readonly
        @click.native="keyboardVisible = !keyboardVisible"
        required
        placeholder="请输入车牌,挂车请输挂车号"
      ></x-input>

      <selector title="交易类型" v-model="changeType" :options="changeTypeList"></selector>
      <selector
        v-show="changeType=='交易'"
        title="品种大类"
        v-model="goodsAllType"
        :options="goodsAllTypeList"
        @on-change="typeChange"
      ></selector>
      <selector
        v-show="goodsAllType=='水果'&&changeType=='交易'"
        title="交易区域"
        v-model="changeArea"
        :options="changeAreaList"
      ></selector>
      <x-number title="车辆吨位" min="0" v-model="dun" fillable></x-number>
      <!--  <selector title="品种" v-model="goodsType" :options="goodTypeList"></selector>  -->
      <cell title="品种" :value="goodsType" is-link="true" @click.native="showSelect"></cell>
      <x-input
        title="驾驶员联系电话"
        v-model="number"
        keyboard="number"
        is-type="china-mobile"
        placeholder="请输入驾驶员电话"
      ></x-input>
      <x-address
        title="产地"
        v-model="address"
        :list="addressData"
        @on-hide="getPlant"
        placeholder="请选择产地"
      ></x-address>
      <x-input title="种植基地" v-model="plant"></x-input>
      <x-number title="产品吨位" min="0" v-model="goodsDun" fillable></x-number>
      <datetime v-model="appointTime" format="YYYY-MM-DD HH:mm" title="预计进场时间"></datetime>
      <x-textarea title="备注" v-model="remark"></x-textarea>
    </group>
    <popup v-model="keyboardVisible">
      <mixed-keyboard
        :args="{plateNumber, autoComplete: false,keyboardType:2}"
        :callbacks="{oncompleted, onkeypressed}"
      />
    </popup>
    <x-dialog v-model="showSelectType">
      <group>
        <x-input placeholder="用拼音码定位" :value="search" @on-change="searchChange"></x-input>
      </group>
      <scroller lock-x scrollbar-y height="400px" ref="scroller">
        <group>
          <cell
            :title="item.value"
            v-for="(item,index) in showGoodsTypeList"
            @click.native="selectGoodsType(item.value)"
            v-bind:key="index"
          ></cell>
        </group>
      </scroller>
    </x-dialog>
    <group>
      <x-button type="primary" @click.native="save">保存</x-button>
    </group>
  </div>
</template>

<script>
import ChinaAddressV4Data from "./china_address.json";
import {
  Group,
  Cell,
  XHeader,
  CheckIcon,
  Search,
  XInput,
  Checker,
  CheckerItem,
  Divider,
  XSwitch,
  XTextarea,
  XButton,
  Selector,
  XAddress,
  Datetime,
  XNumber,
  Value2nameFilter as value2name,
  XDialog,
  Scroller,
  Popup
} from "vux";

import HttpUtil from "../util/HttpUtil";
import { MixedKeyboard } from "vehicle-keyboard";
let pinyin = require("js-pinyin");
export default {
  components: {
    Group,
    Cell,
    XHeader,
    CheckIcon,
    Search,
    XInput,
    Checker,
    CheckerItem,
    Divider,
    XSwitch,
    XTextarea,
    XButton,
    Selector,
    XAddress,
    Datetime,
    XNumber,
    XDialog,
    Scroller,
    MixedKeyboard,
    Popup
  },
  name: "addcar",
  data() {
    return {
      plant: "",
      car: "",
      caruser: "",
      number: "",
      address: [],
      remark: "",
      margin: false,
      goodTypeList: [],
      showGoodsTypeList: [],
      changeType: "",
      goodsAllType: "",
      changeArea: "",
      changeTypeList: ["卸货", "交易"],
      goodsAllTypeList: ["水果", "蔬菜"],
      changeAreaList: ["中央操场", "东关"],
      plateNumber: "",
      dun: 0.0,
      goodsType: "",
      addressData: ChinaAddressV4Data,
      appointTime: "",
      goodsDun: 0.0,
      state: 0,
      showSelectType: false,
      search: "",
      show: true,
      keyboardVisible: false
    };
  },
  mounted() {
    console.log(JSON.stringify(this.$route.params));
    console.log(this.addressData);
    if (this.$route.params.billCode) {
      this.state = 1;
      this.getAppointByBillCode();
    }
    this.getGoodTypeList("");
    this.getAreaList();
  },
  methods: {
    /**
     * 车牌录入完成
     * @param {String} presetNumber 车牌号码
     * @param {Boolean} isAutoCompleted 是否自动完成
     */
    oncompleted(plateNumber, isAutoCompleted) {
      this.plateNumber = plateNumber;
    },
    /**
     * 键位按下监听
     * @param {Object} key 键位对象
     */
    onkeypressed(key) {
      if (key.FUN_OK) {
        this.keyboardVisible = !this.keyboardVisible;
      }
    },

    typeChange(value) {
      console.log(value);
      this.getGoodTypeList(value);
    },
    showSelect() {
      this.showGoodsTypeList = this.goodTypeList;
      this.showSelectType = true;
      this.search = "";
    },
    searchChange(value) {
      this.showGoodsTypeList = this.goodTypeList.filter(function(item) {
        let pycode = pinyin.getCamelChars(item.value);

        return (
          pycode.indexOf(value.toUpperCase()) != -1 ||
          item.value.indexOf(value) != -1
        );
      });
    },
    selectGoodsType(value) {
      this.goodsType = value;
      this.showSelectType = false;
    },
    getPlant(flag) {
      let _this = this;
      if (flag) {
        HttpUtil.get(
          "wechat/getPlantByAddress.do",
          {
            client: this.$store.state.vux.snsUserInfo.username,
            address: value2name(this.address, ChinaAddressV4Data)
          },
          function(data) {
            if (data) {
              _this.plant = data;
            }
          }
        );
      }
    },
    getAppointByBillCode() {
      let _this = this;
      HttpUtil.get(
        "wechat/getAppointByBillCode.do",
        { billCode: this.$route.params.billCode },
        function(data) {
          _this.plateNumber = data.plateNumber;
          _this.dun = parseFloat(data.dun);
          _this.goodsType = data.goodsType;
          _this.number = data.number;
          _this.address = data.address.split(" ");
          _this.goodsDun = parseFloat(data.goodsDun);
          _this.remark = data.remark;
          _this.changeType = data.changeType;
          _this.changeArea = data.changeArea;

          _this.goodsAllType = data.goodsAllType;
          _this.appointTime = data.appointTime;
          _this.plant = data.plant;
        }
      );
    },
    getGoodTypeList(allType) {
      let _this = this;
      HttpUtil.get(
        "wechat/getGoodsTypeList.do",
        {
          allType: allType,
          client: this.$store.state.vux.snsUserInfo.username
        },
        function(data) {
          _this.goodTypeList = data;
          _this.showGoodsTypeList = data;
        }
      );
    },
    getAreaList() {
      let _this = this;
      HttpUtil.get("wechat/getAreaList.do", {}, function(data) {
        _this.changeAreaList = [];
        data.forEach(element => {
          _this.changeAreaList.push(element["交易区域"].trim());
        });
      });
    },
    save() {
      console.log(this.goodsType);
      if (!this.plateNumber) {
        this.$vux.alert.show({
          title: "提示",
          content: "请输入车牌号"
        });
        return;
      }
      if (!this.plant) {
        this.$vux.alert.show({
          title: "提示",
          content: "请输入种植基地"
        });
        return;
      }

      if (!this.dun) {
        this.$vux.alert.show({
          title: "提示",
          content: "请输入车辆吨位"
        });
        return;
      }
      if (!this.goodsType) {
        this.$vux.alert.show({
          title: "提示",
          content: "请输入品种"
        });
        return;
      }
      if (!this.number) {
        this.$vux.alert.show({
          title: "提示",
          content: "请输入驾驶员联系电话"
        });
        return;
      }
      if (this.address.length == 0) {
        this.$vux.alert.show({
          title: "提示",
          content: "请输入产地"
        });
        return;
      }
      if (!this.goodsDun) {
        this.$vux.alert.show({
          title: "提示",
          content: "请输入产品吨位"
        });
        return;
      }
      if (!this.appointTime) {
        this.$vux.alert.show({
          title: "提示",
          content: "请选择预约进场时间"
        });
        return;
      }
      if (!this.$store.state.vux.snsUserInfo.username) {
        this.$vux.alert.show({
          title: "提示",
          content: "页面状态不对，请关闭重新进入"
        });
        return;
      }
      if (!this.changeType) {
        this.$vux.alert.show({
          title: "提示",
          content: "请选择交易类型"
        });
        return;
      }
      if (this.changeType == "交易" && !this.goodsAllType) {
        this.$vux.alert.show({
          title: "提示",
          content: "请选择品种大类"
        });
        return;
      }
      if (
        this.changeType == "交易" &&
        this.goodsAllType == "水果" &&
        !this.changeArea
      ) {
        this.$vux.alert.show({
          title: "提示",
          content: "请选择交易区域"
        });
        return;
      }
      let _this = this;
      let postAppoint = {
        plateNumber: this.plateNumber,
        dun: this.dun,
        goodsType: this.goodsType,
        number: this.number,
        address: value2name(this.address, ChinaAddressV4Data),
        goodsDun: this.goodsDun,
        remark: this.remark,
        plant: this.plant,
        changeType: this.changeType,
        changeArea:
          this.changeType == "交易" && this.goodsAllType == "水果"
            ? this.changeArea
            : "",
        goodsAllType: this.changeType == "交易" ? this.goodsAllType : "",
        appointTime: this.appointTime,
        client: this.$store.state.vux.snsUserInfo.username,
        billCode: this.$route.params.billCode ? this.$route.params.billCode : ""
      };
      HttpUtil.post(
        "wechat/addAppoint.do",
        {
          appoint: JSON.stringify(postAppoint),
          state: this.state
        },
        function(data) {
          if (data.state == 0) {
            _this.$vux.alert.show({
              title: "提示",
              content:
                "该车已被" +
                data.company +
                "预约,预计进场时间是:" +
                data.enterTime
            });
            return;
          }
          if (data.state == -1) {
            _this.$vux.alert.show({
              title: "提示",
              content: "您的ic卡余额不足，请先充值后再预约"
            });
            return;
          } else if (data.state > 0) {
            _this.$vux.toast.show({
              text: _this.state == 0 ? "新增成功" : "修改成功"
            });
            _this.$router.back();
          }
        }
      );
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1,
h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}

.demo1-item {
  border: 1px solid #ececec;
  padding: 5px 15px;
}
.demo1-item-selected {
  border: 1px solid green;
}
.box {
  padding: 0 15px;
}
.header {
  background-color: #1aad19;
}
</style>
