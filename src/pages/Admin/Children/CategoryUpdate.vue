<template>
  <div class="goods-detail">
    <div class="goods-detail-top">分类信息</div>
    <div class="goods-detail-group">
      <div class="goods-item">
        <span>ID</span>
        <span>{{ goodsInfo.cate_id }}</span>
      </div>
      <div class="goods-item">
        <span>名称</span>
        <el-input
          type="text"
          placeholder="请输入内容"
          v-model="goodsInfo.cate_name"
          clearable
          style="width:350px"
        >
        </el-input>
      </div>
      <button @click="saveGoodsInfo()">保存分类信息</button>
    </div>
  </div>
</template>

<script>
import { mapState } from "vuex";
import { changeCateInfo, getAllgoods } from "./../../../api/index";

export default {
  data() {
    return {
      goodsInfo: {},
      options: [
        {
          value: 1,
          label: "图书、音像"
        },
        {
          value: 2,
          label: "家居生活"
        },
        {
          value: 3,
          label: "服饰、箱包"
        },
        {
          value: 4,
          label: "电子产品"
        },
        {
          value: 5,
          label: "美食宝典"
        }
      ]
    };
  },
  mounted() {
    this.goodsInfo = JSON.parse(window.localStorage.getItem("cateInfo"));
  },
  methods: {
    // 修改分类信息
    async saveGoodsInfo() {
      let result = await changeCateInfo(this.goodsInfo);
      if (result.success_code === 200) {
        this.$message({
          type: "success",
          message: "修改成功"
        });
        this.$router.replace("/admin/admincategorys");
        // getAllgoods();
      }
    }
  }
};
</script>

<style scoped lang="stylus" ref="stylesheet/stylus">
.goods-detail
  width 70%
  height 100%
  margin 20px auto
  .goods-detail-top
    width 100%
    height 60px
    line-height 60px
    padding-left 10px
    font-weight bold
  .goods-detail-group
    .goods-icon
      height 60px
      padding 0 10px
      background-color #fff
      border-bottom: 1px solid #e0e0e0
      display flex
      justify-content space-between
      align-items center
      img
        width 50px
        border-radius 50%
    .goods-item
      height 50px
      padding 0 10px
      background-color #fff
      border-bottom: 1px solid #e0e0e0
      display flex
      justify-content space-between
      align-items center
      input
        border 1px solid #ccc
        outline none
        text-align right
        width 200px
    button
      width 20%
      height 40px
      line-height 40px
      background-color #e9232c
      text-align center
      margin 60px 0
      border none
      font-size 16px
      color #fff
      border-radius 10px
  .right-title-color
    color #999
    font-size 14px
</style>
