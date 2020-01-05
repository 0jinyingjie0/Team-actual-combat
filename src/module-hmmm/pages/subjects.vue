<template>
  <el-card>
    <!-- 头部 -->
    <el-row slot="header">
      <el-button type="primary" @click="addItem">新增学科</el-button>
    </el-row>
    <!-- 表单 -->
    <el-form inline label-width="80px">
      <el-form-item label="学科名称">
        <el-input v-model="subjectName"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="clear">清除</el-button>
        <el-button @click="search" type="primary">搜索</el-button>
      </el-form-item>
    </el-form>
    <!-- 表格 -->
    <el-table :data="list">
      <el-table-column prop="id" label="序号"></el-table-column>
      <el-table-column prop="subjectName" label="学科名称"></el-table-column>
      <el-table-column prop="creator" label="创建者"></el-table-column>
      <el-table-column prop="addDate" label="创建日期" width="180px">
        <template slot-scope="obj">{{obj.row.addDate | parseTimeByString}}</template>
      </el-table-column>
      <el-table-column prop="isFrontDisplay" label="前台是否显示">
        <template slot-scope="obj">{{obj.row.isFrontDisplay === 1 ? '显示' : '-'}}</template>
      </el-table-column>
      <el-table-column prop="twoLevelDirectory" label="二级目录"></el-table-column>
      <el-table-column prop="tags" label="标签"></el-table-column> 
      <el-table-column prop="totals" label="题目数量"></el-table-column>
      <el-table-column label="操作" width="220px">
        <template slot-scope="obj">
          <el-button type="text" size="small">学科分类</el-button>
          <el-button type="text" size="small">学科标签</el-button>
          <el-button type="text" size="small" @click="modify(obj.row.id)">修改</el-button>
          <el-button type="text" size="small" @click="delItem(obj.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-row type="flex" justify="end" style="height:60px" align="middle">
      <el-pagination
        background
        layout="prev,pager,next"
        :current-page="page.currentPage"
        :page-size="page.pageSize"
        :total="page.total"
        @current-change="changePage"
      ></el-pagination>
    </el-row>
    <el-dialog :show-close="false" :visible="dialogVisible" @close="beforeClose">
      <el-form ref="myForm" :model="formData" :rules="rules" label-width="150px">
        <el-form-item label="学科名称" prop="subjectName">
          <el-input v-model="formData.subjectName"></el-input>
        </el-form-item>
        <el-form-item label="是否前台显示" prop="isFrontDisplay">
          <el-switch v-model="isFront">是否前台显示</el-switch>
        </el-form-item>
      </el-form>
      <el-row slot="footer" type="flex" justify="end">
        <el-button type="primary" @click="btnOK">确定</el-button>
        <el-button @click="beforeClose">取消</el-button>
      </el-row>
    </el-dialog>
  </el-card>
</template>

<script>
import { list, add, detail, remove, update } from "../../api/hmmm/subjects";

export default {
  name: "SubjectsList",
  data() {
    return {
      dialogVisible: false,
      scienceName: "", // 学科名称
      list: [], // 绑定表格数据
      page: {
        currentPage: 1, // 当前页码
        pageSize: 10, // 每页条数
        total: 0 // 总条数
      },
      subjectName: "",
      formData: {
        subjectName: "",
        isFrontDisplay: 1
      },
      rules: {
        subjectName: [{ required: true, message: "学科名称不能为空" }]
      }
    };
  },
  computed: {
    isFront: {
      get() {
        return this.formData.isFrontDisplay === 1;
      },
      set(value) {
        this.formData.isFrontDisplay = value ? 1 : 0;
      }
    }
  },
  methods: {
    async modify(id) {
      let res = await detail({ id });
      this.formData = res.data;
      this.dialogVisible = true;
    },
    async delItem(id) {
      await this.$confirm("确定删除吗");
      await remove({ id });
      this.getSubject();
    },
    beforeClose() {
      this.resetFields();
      this.dialogVisible = false;
    },
    resetFields() {
      this.formData = { subjectName: "", isFrontDisplay: true };
      this.$refs.myForm.clearValidate();
    },
    addItem() {
      this.dialogVisible = true;
    },
    async btnOK() {
      await this.$refs.myForm.validate();
      this.formData.id ? await update(this.formData) : await add(this.formData);
      this.$message({
        type: "success",
        message: " 操作成功"
      });
      this.getCondition();
      this.resetFields();
      this.dialogVisible = false;
    },
    changePage(newPage) {
      this.page.currentPage = newPage;
      this.getCondition();
    },
    getCondition() {
      let params = {
        page: this.page.currentPage,
        pageSize: this.page.pageSize,
        subjectName: this.subjectName
      };
      this.getSubject(params);
    },
    questionFormatter(row, column, cellValue) {
      let res = this.questionType.filter(item => item.value == cellValue);
      return res.length ? res[0].label : "未知";
    },
    search() {
      this.page.currentPahe = 1;
      this.getCondition();
    },
    clear() {
      this.subjectName = "";
    },
    async getSubject(data) {
      let res = await list(data);
      this.list = res.data.items;
      this.page.total = res.data.counts;
    }
  },
  async created() {
    this.getCondition();
  }
};
</script>

<style scoped>
</style>
