<template>
  <div>
    <swipeout>
    <swipeout-item class="weui-media-box weui-media-box_text" v-for="item in list" transition-mode="follow" :key="item.goodCode">
        <div slot="right-menu">
          <swipeout-button  type="primary" @click.native="onSwapClick(item.goodCode)">历史</swipeout-button>

        </div>
        <div slot="content">
          <div class="weui-media-box_appmsg">
            <div class="weui-media-box__hd" v-if="showImg&&item.src">
              <img class="weui-media-box__thumb" :src="item.src" alt="" @click.prevent="onImgClick(item.bigSrc)">
            </div>
            <div class="weui-media-box__bd" @click.prevent="onItemClick(item)">
              <h4 class="weui-media-box__title" v-html="item.stockGoodsName"></h4>
              <p class="weui-media-box__desc" style="color:red;margin-bottom: 3px">
                ￥
                <span v-if="item.discount>0" class="discount">{{item.discount}}</span>
                {{item.stockGoodsPrice}}
              </p>
              <p class="weui-media-box__desc" v-html="item.spec"></p>
              <x-number style="position:absolute;top:30px;border:none;right:10px" fillable v-model="item.number" :min="0"></x-number>

            </div>
          </div>
        </div>
    </swipeout-item>
    </swipeout>
  </div>
</template>

<script>
import { XNumber, Swipeout, SwipeoutItem, SwipeoutButton, XButton } from "vux";
export default {
  name: "goodsList",
  components: {
    XNumber,
    Swipeout,
    SwipeoutItem,
    SwipeoutButton,
    XButton
  },

  props: {
    list: Array,
    showImg: Boolean
  },

  methods: {
    onItemClick(item) {
      this.$emit("on-click-item", item);
    },
    onImgClick(src) {
      this.$emit("on-click-img", src);
    },
    findbyindex(currentValue) {
      return (currentValue.index = this.index);
    },
    onSwapClick(goodCode) {
      this.$emit("on-swap-click", goodCode);
    }
  }
};
</script>
<style scoped>
.weui-cell:before {
  border: none !important;
}
.discount {
  text-decoration: line-through;
}
</style>

