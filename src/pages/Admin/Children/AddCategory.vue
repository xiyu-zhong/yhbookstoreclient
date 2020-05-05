<template>
  <div id="addgoods">
    <h3>分类上架</h3>
    <el-form
      :model="ruleForm"
      :rules="rules"
      ref="ruleForm"
      label-width="100px"
      class="demo-ruleForm"
    >
      <el-form-item label="分类ID" prop="cate_id" style="width:250px">
        <el-input v-model="ruleForm.cate_id" type="number"></el-input>
      </el-form-item>
      <el-form-item label="分类名称" prop="cate_name" style="width:500px">
        <el-input v-model="ruleForm.cate_name"></el-input>
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
import { addCategory, getAllcategorys } from "./../../../api/index";

export default {
  data() {
    return {
      fileList: [],
      dialogImageUrl: "",
      dialogVisible: false,
      ruleForm: {
        cate_id: null,
        cate_name: "",
      },
      rules: {
        cate_id: [{ required: true, message: "请输入分类ID", trigger: "blur" }],
        cate_name: [
          { required: true, message: "请输入分类名称", trigger: "blur" }
        ]
      }
    };
  },
  methods: {
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
      if (flag) {
        let formData = new FormData();
        formData.append("cate_id", this.$refs[formName].model.cate_id);
        formData.append("cate_name", this.$refs[formName].model.cate_name);

        // let result = await addGoodsToRecom(formData);
        let result = await addCategory(formData);
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
          this.$router.replace("/admin/admincategorys");
        } else if (result.success_code === 500) {
          this.$message({
            type: "info",
            message: "分类已存在"
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
