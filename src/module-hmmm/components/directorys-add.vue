<template>
  <span style="margin-right:10px">
    <el-button type="primary" @click="dialogVisible = true">新增目录</el-button>
    <el-dialog title="提示" :visible.sync="dialogVisible" width="550px" :before-close="handleClose">
      <el-form :model="formData" :rules="formRules" style="margin-left:50px" label-width="100px">
        <el-form-item label="目录名称" prop="directoryName">
          <el-input v-model="formData.directoryName" style="width:300px;margin:0 15px"></el-input>
        </el-form-item>
        <el-form-item label="学科" prop="subjectID">
          <el-select v-model="formData.subjectID" style="width:300px;margin:0 15px">
            <el-option
              v-for="item in option"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            ></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="diradd">确 定</el-button>
      </span>
    </el-dialog>
  </span>
</template>

<script>
import { simple as subjects } from "../../api/hmmm/subjects";
export default {
  data() {
    return {
      formRules: {
        // 校验规则对象 min  max
        directoryName: [{ required: true, message: '标题内容不能为空' }, {
          min: 5, max: 30, message: '标题长度需要在5到30字符之间'
        }]
      },
      formData: {
        directoryName: "",
        subjectID: "", 
      },
      option: [],
      dialogVisible: false
    };
  },
  methods: {
    diradd(){
     this.dialogVisible=!this.dialogVisible
     this.$emit('diradd',this.formData)
    },
    // 学科简单列表
    simple() {
      subjects().then(result => {
        this.option = result.data;
        console.log(result.data);
      });
    },
    open() {
      this.$alert("这是一段内容", "标题名称", {
        confirmButtonText: "确定",
        callback: action => {}
      });
    },
    handleClose(done) {
      this.$confirm("确认关闭？")
        .then(_ => {
          done();
        })
        .catch(_ => {});
    }
  },
  created() {
    this.simple();
  }
};
</script>

<style scoped>
.dir {
  display: flex;
  justify-content: center;
}
</style>
