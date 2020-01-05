<template>
  <el-card>
    <el-row slot="header">
      <el-button type="primary" @click="addItem">新增面试技巧</el-button>
    </el-row>
    <el-form inline label-width="80px">
      <el-form-item label="关键字">
        <el-input v-model="keyword" @change="blur"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="clear">清除</el-button>
        <el-button @click="search" type="primary">搜索</el-button>
      </el-form-item>
    </el-form>
    <el-table :data="list">
      <el-table-column label="序号" prop="id"></el-table-column>
      <el-table-column label="标题" prop="title"></el-table-column>
      <el-table-column label="阅读数" prop="reads"></el-table-column>
      <el-table-column :formatter="formatterBool" label="状态" prop="state">
        <template slot-scope="obj">{{ obj.row.state == 1 ? '启用' :'禁用' }}</template>
      </el-table-column>
      <el-table-column label="创建者" prop="creator"></el-table-column>
      <el-table-column label="操作" width="220px">
        <template slot-scope="obj">
          <el-button type="text" size="small">预览</el-button>
          <el-button type="text" size="small" @click="modify(obj.row.id)">修改</el-button>
          <el-button type="text" size="small" @click="delItem(obj.row.id)">删除</el-button>
          <el-button type="text" size="small" @click="changeState(obj.row)">
            {{
            obj.row.state !==1 ? '启用' :'禁用'
            }}
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-row type="flex" justify="end" style="height:60px" align="middle">
      <el-pagination
        background
        :page-size="page.pageSize"
        :current-page="page.currentPage"
        layout="total, prev, pager, next"
        :total="page.total"
        @current-change="changePage"
      ></el-pagination>
    </el-row>
    <el-dialog :show-close="false" :visible="dialogVisible" @close="beforeClose">
      <el-form ref="myForm" :model="formData" :rules="rules" label-width="150px">
        <el-form-item label="学科名称" prop="title">
          <el-input v-model="formData.title"></el-input>
        </el-form-item>
        <el-form-item prop="articleBody" label="内容">
          <quill-editor
            style="width:100%;height:300px;margin-bottom:120px"
            v-model="formData.articleBody"
          ></quill-editor>
        </el-form-item>
        <el-form-item prop="videoURL" label="视频地址">
          <el-input v-model="formData.videoURL"></el-input>
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
import Vue from "vue"
import {
  list,
  add,
  remove,
  update,
  state,
  detail
} from "../../api/hmmm/articles";
import { quillEditor } from "vue-quill-editor";
import "quill/dist/quill.core.css";
import "quill/dist/quill.snow.css";
import "quill/dist/quill.bubble.css";
Vue.component('quill-editor',quillEditor)

export default {
  data() {
    return {
      dialogVisible: false,
      list: [],
      keyword: "",
      formData: {
        title: "",
        videoURL: "",
        articleBody: ""
      },
      rules: {
        title: [
          { required: true, message: "文章标题不能为空", trigger: "blur" }
        ],
        articleBody: [
          { required: true, message: "文章内容不能为空", trigger: "blur" }
        ],
        videoURL: [
          { required: true, message: "视频地址不能为空", trigger: "blur" }
        ]
      },
      page: {
        currentPage: 1,
        pageSize: 10,
        total: 0
      }
    };
  },
  methods: {
    // 输入框为空时，重新加载数据
    blur() {
      this.getarticles();
    },
    beforeClose() {
      this.resetFields();
      this.dialogVisible = false;
    },
    async modify(id) {
      let result = await detail({ id });
      this.formData = result.data;
      this.dialogVisible = true;
    },
    async changeState(row) {
      await this.$confirm("确认改变状态吗");
      await state({ id: row.id, state: row.state === 1 ? 0 : 1 });
      this.getarticles();
    },
    async delItem(id) {
      await this.$confirm("确认删除此目录吗");
      await remove({ id });
      this.getarticles();
    },
    resetFields() {
      // this.$refs.myForm.resetFields();
      this.formData = {
        articlesName: "",
        subjectID: null
      };
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
        message: "操作成功"
      });
      this.resetFields();
      this.getCondition();
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
        keyword:this.keyword
      };
      this.getarticles(params);
    },
    questionFormatter(row, column, cellValue) {
      let result = this.questionType.filter(item => item.value == cellValue);
      return result.length ? result[0].label : "未知";
    },
    search() {
      this.page.currentPage = 1;
      this.getCondition();
    },
    clear() {
      this.articlesName = "";
      this.getarticles();
    },
    async getarticles(data) {
      let result = await list(data);
      this.list = result.data.items;
      this.page.total = result.data.counts;
    },
    formatterBool(row, column, cellValue) {
      return cellValue ? "正常" : "关闭";
    }
  },

  async created() {
    this.getarticles();
  }
};
</script>

<style lang="scss"></style>
