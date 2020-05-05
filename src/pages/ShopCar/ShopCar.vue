<template>
  <div id="shopcar">
    <div id="content">
      <div class="cont_title">
        <span>购物清单</span>
      </div>
      <div class="cont_op">
        <div class="con_selet">
          <input
            type="checkbox"
            class="con_selectAll"
            :checked="isSelectedAll"
            @click.stop="selectedAll(isSelectedAll)"
          />
          <label for="con_selectAll">全选</label>
        </div>
        <div>书籍图片</div>
        <div>书籍信息</div>
        <div>书籍单价</div>
        <div>书籍数量</div>
        <div>书籍金额</div>
        <div>操作</div>
      </div>
      <div class="pro_box" v-if="cartgoods.length">
        <div class="goods" v-for="(goods, index) in cartgoods" :key="index">
          <ul class="item_content">
            <li class="td td-chk">
              <div>
                <input
                  type="checkbox"
                  class="checkBox"
                  :checked="goods.checked"
                  @click="singleSelected(goods)"
                />
              </div>
            </li>
            <li class="td td-item">
              <div class="td-item-pic">
                <img :src="goods.thumb_url" />
              </div>
            </li>
             <li class="td td-price">
                <div class="td-item-info">{{ goods.goods_name }}</div>
            </li>
            <li class="td td-price">
              <strong>{{ (goods.price) | moneyFormat }}</strong>
            </li>
            <li class="td td-amount">
              <div class="item-amout">
                <el-input-number
                  v-model="goods.buy_count"
                  :min="1"
                  :max="goods.counts"
                  @change="updateGoodsCount(goods, goods.buy_count)"
                ></el-input-number>
              </div>
            </li>
            <li class="td td-sum">
              <strong>{{
                ((goods.buy_count * goods.price)) | moneyFormat
              }}</strong>
            </li>
            <li class="td td-op">
              <a @click="clickTrash(goods)">删除</a
              >  &nbsp;&nbsp;  &nbsp;<a @click="addToOrder(goods)">下单</a>
            </li>
          </ul>
        </div>
      </div>
      <div class="pro_box" v-else>购物车里空空如也</div>
      <div id="footer">
      <div class="foot_select">
        <input
          type="checkbox"
          class="foot_selectAll"
          name="foot_selectAll"
          :checked="isSelectedAll"
          @click.stop="selectedAll(isSelectedAll)"
        />
        <label for="foot_selectAll">全选</label>
      </div>
      <div class="foot_op">
        <a class="foot_remove" @click.prevent="emptyCart">清空购物车</a>
      </div>
      <div class="foot_total">
        <div class="amout-sum">
          <span class="txt">已选书籍</span>
          <strong id="selected_amout">{{ totalAmount }}</strong>
          <span class="txt">件</span>
        </div>
        <div class="price-sum">
          <span class="txt">合计（不含运费）：</span>
          <strong class="selected_price">{{
            totalPrice | moneyFormat(totalPrice)
          }}</strong>
        </div>
        <div class="btn-area">
          <a
            class="btn-sumbit"
            :class="{ 'btn-allow': totalAmount }"
            @click="handleToOrder"
            >结&nbsp;算</a
          >
        </div>
      </div>
    </div>
    </div>
    
  </div>
</template>

<script>
import { mapState } from "vuex";
import { mapActions } from "vuex";
import { Message } from "element-ui";
import { postOrder } from "../../api";

export default {
  data() {
    return {
      shopsNum: [],
      isSelectedAll: false,
      totalPrice: 0, // 书籍总价
      currentDelGoods: {}, // 当前删除的书籍
      totalAmount: 0
    };
  },
  computed: {
    ...mapState(["userInfo", "cartgoods"])
  },
  mounted() {
    let user_id = this.userInfo.id;
    // 请求书籍数据
    this.$store.dispatch("reqCartsGoods", { user_id });
  },
  methods: {
    handleToOrder() {},
    async addToOrder(goods) {
      const order = {
        id: goods.id,
        num: goods.buy_count,
        price: goods.price,
        total: parseInt(goods.buy_count) * parseInt(goods.price),
        g_id: goods.goods_id,
        u_id: this.userInfo.id
      };
      const ret = await postOrder(order);
      if (ret) {
        Message.success("下单成功");
        let user_id = this.userInfo.id;
        this.currentDelGoods = goods;
        this.$store.dispatch("delSingleGoods", { goods, user_id });
        this.getAllGoodsPrice();
      }
    },
    // 1.更新单个书籍数量
    updateGoodsCount(goods, count) {
      let user_id = this.userInfo.id;
      this.$store.dispatch("updateGoodsCount", { goods, count, user_id });
      this.getAllGoodsPrice();
    },
    // 2.是否选中所有书籍
    selectedAll(isSelectedAll) {
      this.isSelectedAll = !isSelectedAll;
      this.$store.dispatch("selectedAll", { isSelectedAll });
      this.getAllGoodsPrice();
    },
    // 3.计算书籍总价
    getAllGoodsPrice() {
      let totalPrice = 0;
      this.cartgoods.forEach((goods, index) => {
        if (goods.checked) {
          totalPrice += (goods.price) * goods.buy_count;
        }
      });
      this.totalPrice = totalPrice;
    },
    // 4.单个书籍的选中与否
    singleSelected(goods) {
      this.$store.dispatch("singleSelected", { goods });
      this.hasSelectedAll();
      this.getAllGoodsPrice();
    },
    // 5.判断是否全选
    hasSelectedAll() {
      let flag = true;
      let totalNum = 0;
      this.cartgoods.forEach((goods, index) => {
        if (!goods.checked) {
          flag = false;
        } else {
          totalNum += 1;
        }
      });
      this.totalAmount = totalNum;
      this.isSelectedAll = flag;
    },
    // 6.删除单个书籍
    clickTrash(goods) {
      this.$confirm("您确定删除该书籍吗？", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          let user_id = this.userInfo.id;
          this.currentDelGoods = goods;
          this.$store.dispatch("delSingleGoods", { goods, user_id });
          this.getAllGoodsPrice();
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
    // 7.删除所有书籍
    emptyCart() {
      this.$confirm("您确定删除所有书籍吗？", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          let user_id = this.userInfo.id;
          this.$store.dispatch("delAllGoods", { user_id });
          this.getAllGoodsPrice();
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    }
  }
};
</script>

<style scoped>
#shopcar{
  border: 2px solid #dbdbdb;
  border-radius: 12px;
  margin: 20px;
}
#content > .cont_title {
  margin: 0 auto;
  width: 100%;
  line-height: 40px;
  height: 50px;
  border-bottom: 2px solid #dbdbdb;
}
.cont_title > span {
  display: inline-block;
  font-weight: 700;
  font-size: 17px;
  height: 100%;
  width: 100px;
  text-align: center;
  line-height: 25px;
  cursor: pointer;
  color: rgba(0,0,0,.6);
  }
#content > .cont_op {
  width: 100%;
  margin: 0 auto;
  height: 40px;
  display: flex;
  justify-content: space-around;
  background: #f5f5f5;
}
.cont_op > div {
  line-height: 40px;
}
.cont_op > div:nth-child(2) {
  width: 120px;
}
#content > .pro_box {
  margin: 0 auto;
  width: 100%;
}
.pro_box > .goods {
  background: #fcfcfc;
  width: 100%;
  height: 130px;
  border-radius: 0 0 12px 12px;
}
.goods > .item_content {
  padding: 16px 0 16px 16px;
  list-style: none;
  display: flex;
  justify-content: space-around;
  border-bottom: 1px solid #f0f0f0;
}
.item_content > .td-item {
  width: 180px;
  position: relative;
  text-align: center;
}
.td-item > div {
  display: inline-block;
  text-align: center;
}
.td-item > .td-item-pic > img {
  width: 80px;
  height: 80px;
  margin: 0 auto;
}
.td-item > .td-item-info {
  position: absolute;
  width: 80px;
  line-height: 15px;
  font-size: 12px;
  top: 0;
}
.item_content > .td-sum {
  color: #d44d44;
}
.td-amount > .item-amout {
  height: 25px;
}
.item-amout a {
  display: inline-block;
  height: 23px;
  width: 17px;
  border: 1px solid #e5e5e5;
  background: #f0f0f0;
  text-align: center;
  line-height: 23px;
  color: #444;
  cursor: pointer;
}
.item-amout a:hover {
  color: #f50;
  border-color: #f60;
}
.item-amout > .text_amount {
  width: 40px;
  height: 20px;
  text-align: center;
  display: inline-block;
}
.td-op a {
  color: black;
}
.td-op a:hover {
  cursor: pointer;
  color: #d44d44;
}
#footer {
  width: 100%;
  height: 50px;
  background: #e5e5e5;
  border-radius:0 0 12px 12px;
}
#footer div {
  display: inline-block;
}
#footer > .foot_select {
  width: 50px;
  height: 50px;
  line-height: 50px;
  padding-left: 5px;
  cursor: pointer;
  font-size: 14px;
}
.foot_select input,
.foot_select label {
  cursor: pointer;
}
.foot_op {
  line-height: 50px;
  height: 50px;
}
.foot_op > .foot_remove {
  margin-left: 25px;
  font-size: 14px;
  cursor: pointer;
}
.foot_remove:hover {
  color: #d44d44;
}
.foot_total {
  float: right;
}
.foot_total > .amout-sum {
  cursor: pointer;
  height: 50px;
  color: #3c3c3c;
}
.foot_total .txt {
  line-height: 50px;
  font-size: 14px;
}
.amout-sum > #selected_amout {
  padding: 0 5px;
  color: #d44d44;
  font-weight: 700;
  font-size: 18px;
}
.price-sum {
  margin-left: 50px;
}
.price-sum > .selected_price {
  color: #d44d44;
  font-weight: 700;
  font-size: 20px;
  line-height: 50px;
}
.btn-area > .btn-sumbit {
  display: inline-block;
  background: #b0b0b0;
  color: #fff;
  border-left: 1px solid #e7e7e7;
  width: 119px;
  height: 50px;
  cursor: not-allowed;
  text-align: center;
  line-height: 50px;
  font-size: 20px;
}
.btn-area > .btn-sumbit.btn-allow {
  background: #f22d00;
  cursor: pointer;
}
</style>
