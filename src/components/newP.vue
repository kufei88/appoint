<template>
  <div style="margin-bottom:50px">
    <div v-for="imgItem in imglist" style="background-color:#fff;text-align:center;">
      <span style="font-size:20px;">加载中</span>
      <x-img :src="imgItem.src" :webp-src="`${imgItem.src}?type=webp`" @on-success="success" @on-error="error" class="ximg-demo" error-class="ximg-error" :offset="-100" container="#vux_view_box_body"></x-img>
    </div>
  </div>
</template>
<script>
import { XImg } from "vux";
import HttpUtil from "../util/HttpUtil";
export default {
  components: {
    XImg
  },
  data() {
    return {
      imglist: []
    };
  },
  mounted() {
    this.getImgList();
  },
  methods: {
    getImgList() {
      let _this = this;
      HttpUtil.get("goods/getNewPList.do", {}, function(data) {
        data.forEach(function(item) {
          item.src = "../detailsImg/" + item.goodsCode + ".jpg";
        });
        _this.imglist = data;
        console.log(data);
      });
    },
    success(src, ele) {
      console.log("success load", src);
      const span = ele.parentNode.querySelector("span");
      ele.parentNode.removeChild(span);
    },
    error(src, ele, msg) {
      console.log("error load", msg, src);
      const span = ele.parentNode.querySelector("span");
      span.innerText = "加载失败";
    }
  }
};
</script>

<style>
.ximg-demo {
  width: 100%;
  height: auto;
}
.ximg-error {
  background-color: yellow;
}
.ximg-error:after {
  content: "加载失败";
  color: red;
}
</style>
