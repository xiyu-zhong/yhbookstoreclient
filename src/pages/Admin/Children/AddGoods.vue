<template>
  <div id="addgoods">
    <h3>书籍上架</h3>
    <el-form
      :model="ruleForm"
      :rules="rules"
      ref="ruleForm"
      label-width="100px"
      class="demo-ruleForm"
    >
      <el-form-item label="书籍ID" prop="goods_id" style="width:250px">
        <el-input v-model="ruleForm.goods_id" type="number"></el-input>
      </el-form-item>
      <el-form-item label="书籍名称" prop="short_name" style="width:500px">
        <el-input v-model="ruleForm.short_name"></el-input>
      </el-form-item>
      <el-form-item label="书籍图片" style="width:500px">
        <el-upload
          list-type="picture-card"
          :on-preview="handlePictureCardPreview"
          :on-change="handleChange"
          :auto-upload="false"
          :multiple="false"
          limit:1
          action="http://localhost:3000"
        >
          <i class="el-icon-plus"></i>
        </el-upload>
        <el-dialog :visible.sync="dialogVisible">
          <img width="100%" :src="dialogImageUrl" alt="" />
        </el-dialog>
      </el-form-item>
      <el-form-item label="书籍价格" prop="price" style="width:250px">
        <el-input v-model="ruleForm.price" type="number"></el-input>
      </el-form-item>
      <el-form-item label="书籍库存" prop="counts" style="width:250px">
        <el-input v-model="ruleForm.counts" type="number"></el-input>
      </el-form-item>
      <el-form-item label="书籍分类" prop="category">
        <el-select v-model="ruleForm.category" placeholder="请选择">
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="书籍作者" prop="sales_tip" style="width:500px">
        <el-input v-model="ruleForm.sales_tip"></el-input>
      </el-form-item>
      <el-form-item label="书籍详情" prop="goods_name" style="width:500px">
        <el-input
          type="textarea"
          v-model="ruleForm.goods_name"
          autosize
        ></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="submitForm('ruleForm')"
          >立即创建</el-button
        >
        <el-button @click="resetForm('ruleForm')">重置</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import { mapState } from "vuex";
import {
  addGoodsToRecom,
  getAllgoods,
  getAllCategorys
} from "./../../../api/index";

export default {
  data() {
    return {
      fileList: [],
      dialogImageUrl: "",
      dialogVisible: false,
      ruleForm: {
        goods_id: null,
        short_name: "",
        goods_name: "",
        price: null,
        sales_tip: "",
        counts: null,
        category: 1
      },
      rules: {
        goods_id: [
          { required: true, message: "请输入书籍ID", trigger: "blur" }
        ],
        short_name: [
          { required: true, message: "请输入书籍名称", trigger: "blur" }
        ],
        goods_name: [
          { required: true, message: "请填写书籍详情", trigger: "blur" }
        ],
        price: [{ required: true, message: "请输入书籍价格", trigger: "blur" }],
        counts: [
          { required: true, message: "请输入书籍库存", trigger: "blur" }
        ],
        category: [
          { required: true, message: "请选择书籍分类号", trigger: "blur" }
        ],
        sales_tip: [
          { required: true, message: "请输入书籍作者", trigger: "blur" }
        ]
      },
      options: []
    };
  },
  methods: {
    async getAllGoods1() {
      let result = await getAllCategorys();
      if (result.success_code === 200) {
        // console.log("result", result);
        this.options = result.message.map(item => {
          return {
            value: item.cate_id,
            label: item.cate_name
          };
        });
      }
    },
    handleChange(file, fileList) {
      if (fileList.length > 1) {
        fileList = fileList.slice(-1);
      }
      this.fileList = fileList;
    },
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },
    async submitForm(formName) {
      let flag = false;
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.$refs[formName].model.goods_id = Number(
            this.$refs[formName].model.goods_id
          );
          this.$refs[formName].model.counts = Number(
            this.$refs[formName].model.counts
          );
          this.$refs[formName].model.price = Number(
            this.$refs[formName].model.price
          );
          // console.log(this.$refs[formName].model);
          flag = true;
        }
      });
      if (!this.fileList.length) {
        flag = false;
      }
      if (flag) {
        let formData = new FormData();
        formData.append("goods_id", this.$refs[formName].model.goods_id);
        formData.append("short_name", this.$refs[formName].model.short_name);
        formData.append("goods_name", this.$refs[formName].model.goods_name);
        formData.append("counts", this.$refs[formName].model.counts);
        formData.append("price", this.$refs[formName].model.price);
        formData.append("sales_tip", this.$refs[formName].model.sales_tip);
        formData.append("category", this.$refs[formName].model.category);
        formData.append("goods_img", this.fileList[0].raw);

        let result = await addGoodsToRecom(formData);
        /*
              let result = await addGoodsToRecom({
                goods_id: this.$refs[formName].model.goods_id,
                short_name: this.$refs[formName].model.short_name,
                goods_name: this.$refs[formName].model.goods_name,
                counts: this.$refs[formName].model.counts,
                price: this.$refs[formName].model.price,
                sales_tip: this.$refs[formName].model.sales_tip,
                category: this.$refs[formName].model.category
              });
          */
        if (result.success_code === 200) {
          this.$message({
            type: "success",
            message: "添加成功"
          });
          this.$router.replace("/admin");
          getAllgoods();
        } else if (result.success_code === 500) {
          this.$message({
            type: "info",
            message: "书籍已存在"
          });
        } else {
          this.$message({
            type: "error",
            message: "添加失败"
          });
        }
      }
    },
    resetForm(formName) {
      this.$refs[formName].resetFields();
    }
  },
  async mounted() {
    await this.getAllGoods1();
  }
};
</script>

<style scoped>
#addgoods {
  padding-top: 20px;
}
#addgoods > h3 {
  font-weight: bolder;
  margin: 10px 10px 30px 10px;
}
</style>
