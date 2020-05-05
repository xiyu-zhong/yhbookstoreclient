<template>
  <div class="header_container">
    <div class="header_img header_img_left">
      <img src="../../common/img/yanruyu.png" />
    </div>
    <div class="header_search">
      <div>
        <input type="text" placeholder="搜索  书籍名称" v-model="keywords" />
        <el-button type="info" icon="el-icon-search" @click="postSearch"></el-button>
      </div>
    </div>
    <div class="header_img header_img_right">
      <img src="../../common/img/huangjinwu.png" />
    </div>
  </div>
</template>

<script>
import { MessageBox } from "element-ui";
import { mapState } from "vuex";
import { mapActions } from "vuex";

export default {
  data() {
    return {
      keywords: ""
    };
  },
  computed: {
    ...mapState(["userInfo"])
  },
  methods: {
    goShopCar() {
      if (this.userInfo.id) {
        this.$router.replace("/shopcar");
      } else {
        MessageBox({
          type: "info",
          message: "请先登录!",
          showClose: true
        });
      }
    },
    async postSearch() {
      if (this.keywords) {
        let keywords = this.keywords;
        this.$store.dispatch("reqSearch", { keywords });
        this.$router.replace("/searchdetail");
      }
    }
  }
};
</script>

<style scoped>
/*头部搜索区*/
.header_container {
  position: relative;
  width: 100%;
  height: 140px;
  margin: 0;
  padding: 0;
}
/*商城标识图片区域*/
.header_container > .header_img {
  position: absolute;
  width: 200px;
  height: 200px;
}
.header_img > img {
  width: 100%;
  height: 100%;
}
.header_img_left {
  left: 100px;
}
.header_img_right {
  right: 100px;
}
/*鼠标移入图片实现3d动画*/
.header_img:hover {
  transform-style: preserve-3d;
  transform: rotateY(720deg);
  transition: all 1500ms;
  -moz-transform-style: preserve-3d;
  -moz-transform: rotateY(45deg);
  -moz-transition: all 1500ms;
}
.header_search {
  position: absolute;
  left: 50%;
  margin-left: -290px;
  margin-top: 50px;
}
.header_search div {
  display: flex;
  align-items: center;
}
.header_search input {
  padding-left: 10px;
  width: 490px;
  height: 35px;
  font-size: 14px;
  line-height: 35px;
  border: 2px solid rgba(0,0,0,.08);
  border-right: none;
  outline: none;
  border-radius: 18px 0 0 18px;
}
.header_search button.el-button.el-button--info{
  padding: 0;
  width: 60px;
  height: 35px;
  font-size: 18px;
  border-radius: 0 18px 18px 0;
  outline: none;
  border: 2px solid rgba(0,0,0,.08);
  display: flex;
  justify-content: center;
  align-items: center;
}
.header_search > p > a {
  display: inline-block;
  padding: 5px 10px;
  font-size: 12px;
  color: #999;
  cursor: pointer;
}
.header_search > p > a:hover {
  color: red;
}
</style>
