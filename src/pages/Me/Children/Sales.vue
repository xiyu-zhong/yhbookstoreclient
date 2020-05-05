<template>
  <div>
    <el-table :data="tempData" style="width: 100%">
      <!-- <el-table-column type="expand">
        <template slot-scope="props">
          <el-form label-position="left" inline class="demo-table-expand">
            <el-form-item label="书籍名称">
              <span>{{ props.row.short_name }}</span>
            </el-form-item>
            <el-form-item label="书籍 ID">
              <span>{{ props.row.goods_id }}</span>
            </el-form-item>
            <el-form-item label="书籍分类">
              <span>{{ category[props.row.category - 1] }}</span>
            </el-form-item>
            <el-form-item label="书籍价格">
              <span>{{ props.row.price | priceFormat }}</span>
            </el-form-item>
            <el-form-item label="书籍库存">
              <span>{{ props.row.counts }}</span>
            </el-form-item>
            <el-form-item label="书籍描述">
              <span>{{ props.row.goods_name }}</span>
            </el-form-item>
            <el-form-item label="书籍图片">
              <img :src="props.row.thumb_url" style="width:70px" />
            </el-form-item>
          </el-form>
        </template>
      </el-table-column> -->
      <el-table-column label="订单 ID" prop="id"> </el-table-column>
      <el-table-column label="书籍名称" prop="goods_name"> </el-table-column>
      <el-table-column label="书籍价格" prop="price"> </el-table-column>
      <el-table-column label="书籍数量" prop="num"> </el-table-column>
      <el-table-column label="用户名" prop="user_name"> </el-table-column>
      <el-table-column label="用户地址" prop="user_address"> </el-table-column>
      <el-table-column label="总价" prop="total"> </el-table-column>
      <el-table-column label="状态" prop="state"> </el-table-column>
    </el-table>
  </div>
</template>

<script>
import { getAllorders, deleteOrder, updateOrder } from "./../../../api/index";
import { mapState } from "vuex";
import { mapActions } from "vuex";
import { MessageBox, Message } from 'element-ui';

export default {
  data() {
    return {
      currentIndex: 1,
      pageSize: 5,
      tempData: []
    };
  },
  mounted() {
    const state = this.$route.query.state
    this.state = state
    this.getAllGoods(state);
  },
  methods: {
    async handleEdit(index, row) {
      // 发货
      const ret = await updateOrder({id: row.o_id, state: 1})
      if(ret) {
        Message.success("发货成功")
      }
    },
    async handleDelete(index, row) {
      // console.log("row", row);
      MessageBox.confirm("您确定永久删除该订单吗?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(async () => {
          let result = await deleteOrder(row.o_id);
          if (result.success_code === 200) {
            this.$message({
              type: "success",
              message: "已删除"
            });
            await this.getAllGoods()
          }
        })
        .catch((e) => {
          console.log("e", e);
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
    async getAllGoods(state) {
      let result = await getAllorders(state);
      console.log("result", result);
      if (result.success_code === 200) {
        this.tempData = result.data.map(item => {
          return {
            ...item,
            state: item.state == '0' ? '待发货' : '待收货'
          }
        });
      }
    }
  },
  computed: {
    pageNum() {
      return Math.ceil(this.tableData.length / this.pageSize);
    }
  },
  filters: {
    priceFormat(price) {
      return price.toFixed(2);
    }
  },
  watch: {
    "$route": function() {
      this.getAllGoods(this.$route.query.state)
    }
  }
};
</script>

<style scoped>
.el-table {
  margin: 20px auto 50px;
}
.demo-table-expand {
  font-size: 0;
}
.demo-table-expand label {
  width: 90px;
  color: #99a9bf;
}
.demo-table-expand .el-form-item {
  margin-right: 0;
  margin-bottom: 0;
  width: 50%;
}
.el-pagination {
  text-align: center;
}
</style>
