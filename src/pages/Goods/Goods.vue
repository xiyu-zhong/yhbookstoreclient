<template>
  <div id="container" v-if="goodsDetail[0]">
    <div class="pro_detail">
      <div class="pro_img" :style="{backgroundImage: 'url(' + Carbg + ')' }">
          <pic-zoom :url="goodsDetail[0].image_url" class="pro_middle_img" :scale="3"> </pic-zoom>
      </div>
      <div class="pro_meg">
        <div class="pro_meg_hd">
          <h1>
            {{ goodsDetail[0].goods_name }}
          </h1>
        </div>
        <div class="pro_meg_price">
          <dl>
            <dt>促销价</dt>
            <dd>
              <div class="promo_price">
                <span class="tm-price">{{
                  (goodsDetail[0].price) | moneyFormat
                }}</span>
                <b>疫情期间优惠促销</b>
              </div>
            </dd>
          </dl>
          <dl>
            <dt>市场价</dt>
            <dd class="nor_price">
              {{ (goodsDetail[0].normal_price)/1000+10 | moneyFormat }}
            </dd>
          </dl>
          <dl>
             <dt>书籍作者</dt>
            <dt class="sale_tip">{{ goodsDetail[0].sales_tip }}</dt>
          </dl>
        </div>
        <div class="pro_meg_deliver">
          <dl>
            <dt>自取地点</dt>
            <dd>
              洛阳师范学院枣园宿舍
            </dd>
          </dl>
        </div>
        <div class="pro_meg_console">
          <dl class="tb-sku">
            <dt>数量</dt>
            <dd>
              <div class="item-amout">
                <el-input-number
                  v-model="shopNum"
                  :min="1"
                  :max="goodsDetail[0].counts"
                ></el-input-number>
              </div>
              <span>库存</span><em>{{ goodsDetail[0].counts }}</em
              ><span>件</span>
            </dd>
          </dl>
          <div class="shopping_car">
            <el-button
              type="success"
              @click.prevent="dealWithCellBtnClick(goodsDetail[0])"
              >加入购物车</el-button
            >
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { postComment, addGoodsToCart } from "./../../api/index";
import { MessageBox } from "element-ui";
import { mapActions } from "vuex";
import { mapState } from "vuex";
import carbg from '../../common/img/timg.jpg';
import PicZoom from 'vue-piczoom';
export default {
  data() {
    return {
      textarea: "",
      rating: 0,
      colors: ["#99A9BF", "#F7BA2A", "#FF9900"],
      currentGoodsId: 0,
      shopNum: 1,
      Carbg:carbg
    };
  },
  components:{
    PicZoom
  },
  computed: {
    ...mapState(["goodsDetail", "userInfo", "goodsComment"])
  },
  created() {
    this.currentGoodsId = this.$route.params.id;
    this.$store.dispatch("reqGoodsDetail", {
      goodsNo: this.currentGoodsId
    });
    this.$store.dispatch("reqGoodsComment", {
      goodsId: this.currentGoodsId
    });
  },
  mounted() {
    this.currentGoodsId = Number(this.$route.params.id);
    console.log("this.currentGoodsId", this.currentGoodsId);
    this.$store.dispatch("reqGoodsDetail", {
      goodsNo: this.currentGoodsId
    });
    this.$store.dispatch("reqGoodsComment", {
      goodsId: this.currentGoodsId
    });
    this.$store.dispatch("getUserInfo");
  },
  watch: {
    $route() {
      this.currentGoodsId = this.$route.params.id;
      if (this.currentGoodsId) {
        this.$store.dispatch("reqGoodsDetail", {
          goodsNo: this.currentGoodsId
        });
        this.$store.dispatch("reqGoodsComment", {
          goodsId: this.currentGoodsId
        });
        // 请求当前用户信息
        this.$store.dispatch("getUserInfo");
      }
    }
  },
  methods: {
    // 监听书籍点击
    async dealWithCellBtnClick(goods) {
      // 1. 发送请求
      // user_id, goods_id, goods_name, thumb_url, price, buy_count, counts
      if (this.userInfo.user_name) {
        let result = await addGoodsToCart(
          this.userInfo.id,
          goods.goods_id,
          goods.short_name,
          goods.thumb_url,
          goods.price,
          this.shopNum,
          goods.counts
        );
        if (result.success_code === 200) {
          MessageBox({
            type: "success",
            message: result.message,
            showClose: true
          });
          let user_id = this.userInfo.id;
          // 请求书籍数据
          this.$store.dispatch("reqCartsGoods", { user_id });
          this.shopNum = 1;
        }
      }
    }
  }
};
</script>

<style scoped>
#container > .pro_detail {
  width: 990px;
  position: relative;
  z-index: 100;
  margin: 20px auto;
  display: flex;
}
.pro_detail > .pro_img {
  position: relative;
  width: 480px;
  height: 360px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-repeat: no-repeat;
  background-size: 100%;
}
.pro_middle_img{
  max-width: 200px;
  max-height: 150px;
}
.pro_detail > .pro_meg {
  margin: 0 0 0 20px;
  width: 470px;
  color: #666;
  padding: 0 0 3px;
}
.pro_meg > .pro_meg_hd {
  padding: 0 10px 12px;
  color: #000;
}
.pro_meg_hd > h1 {
  font-size: 16px;
  font-weight: 700;
  color: #67c23a;
}
.pro_meg > .pro_meg_price {
  padding: 5px 20px;
  height: 200px;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
}
.pro_meg_price dl {
  display: flex;
  align-items: center;
  border-bottom: 1px solid rgba(0,0,0,.08);
  margin-bottom: 0 !important;
 padding: 10px 0;
  cursor: pointer;
}
.pro_meg_price dl dt {
  width: 70px;
  color: #999;
  font-size: 12px;
}
.pro_meg_price dl dd {
  margin-bottom: 0 !important;
  font-family: Arial;
}
.pro_meg_price dl dd div {
  display: flex;
  align-items: center;
}
.pro_meg_price dl:last-child dd {
  color: #67c23a;
  font-weight: bold;
  font-size: 12px;
}
.promo_price {
  line-height: 24px;
  vertical-align: middle;
  color: #67c23a;
  font-size: 18px;
  font-family: Arial;
  -webkit-font-smoothing: antialiased;
}
.promo_price b {
  display: inline-block;
  font-weight: normal;
}
.promo_price b:last-child {
  font-size: 12px;
  background: #67c23a;
  color: white;
  padding: 0 6px;
}
.promo_price > .tm-price {
  font-size: 20px;
  display: inline-block;
  margin-right: 12px;
  font-weight: bold;
}
.nor_price {
  text-decoration: line-through;
}
.sale_tip {
  color: #67c23a !important;
  font-weight: bolder;
  width: 80px !important;
}
.pro_meg_deliver {
  margin: 5px 20px -15px 0;
  padding: 5px;
}
.pro_meg_deliver dl {
  padding: 5px;
  font-size: 14px;
  color: black;
  cursor: pointer;
}
.pro_meg_deliver dl dt {
  color: #999;
  font-size: 14px;
  text-align: left;
  width: 69px;
  margin: 0 0 0 8px;
  float: left;
}
.pro_meg_deliver dl dd {
  font-size: 13px;
}
.pro_meg_console {
  margin: 5px 20px 5px 0;
  padding: 5px;
}
.tb-sku {
  padding: 5px;
  font-size: 14px;
  color: black;
  cursor: pointer;
}
.tb-sku dt {
  color: #999;
  font-size: 14px;
  text-align: left;
  width: 69px;
  margin: 0 0 0 8px;
  float: left;
}
.tb-sku dd {
  font-size: 13px;
}
.tb-sku dd div {
  display: inline-block;
  margin-right: 20px;
}
.item-amout {
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
.shopping_car {
  margin: 20px auto 0;

  display: flex;
  justify-content: center;
}
.shopping_car button {
  outline: none;
}
</style>
