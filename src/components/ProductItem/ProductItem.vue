<template>
  <div class="pro" @click="goDetail(pro.goods_id)">
    <img v-lazy="pro.thumb_url" class="pro_img" />
    <div class="pro_text">
      <p>{{ pro.short_name }}</p>
      <p>{{ pro.price | moneyFormat }}</p>
    </div>
  </div>
</template>

<script>
import { mapState } from "vuex";

export default {
  props: {
    pro: {
      type: Object,
      default: {}
    }
  },
  methods: {
    goDetail(id) {
      this.$router.replace("/goods/" + id);
    }
  }
};
</script>

<style scoped>
.pro {
  position: relative;
  width: 25%;
  border-right: 1px solid rgba(0, 0, 0, 0.1);
  height: 368px;
  box-sizing: border-box;
  background: white;
  display: flex;
  flex-direction: column;
}
.pro:last-child{
  border-right: none;
}
.pro > .pro_img {
  margin: 30px auto 20px;
  width: 216px;
  height: 216px;
}
.pro > .pro_text {
  height: 130px;
  line-height: 25px;
}
.pro_text > p {
  margin-bottom: 10px;
  font-family: Helvetica Neue, PingFang SC, Hiragino Sans GB, Heiti SC,
    Microsoft YaHei, WenQuanYi Micro Hei, sans-serif;
}
.pro_text > p:first-child {
  font-size: 14px;
  font-weight: 700;
  text-align: center;
  line-height: 1.2;
  margin: 20px 0;
  width: 100%;
  color: #333;
  white-space: nowrap;
  text-overflow: ellipsis;
  overflow: hidden;
}
.pro_text > p:nth-child(2) {
  font-family: Arial;
  font-size: 18px;
  font-weight: bolder;
  color: #d44d44;
  font-style: normal;
  text-align: center;
}
.pro_text > p:nth-child(3) {
  font-size: 12px;
}
</style>
