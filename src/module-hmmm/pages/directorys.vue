<template>
  <el-card>
    <div slot="header">
      <!-- <el-button type="primary">新增目录</el-button> -->
      <directorysadd @diradd="diradd"></directorysadd>
      <el-button type="primary">返回学科</el-button>
    </div>
    <el-form inline>
      <el-form-item label="目录">
        <el-input v-model="searchForm.directoryName" @change="blur"></el-input>
      </el-form-item>
      <el-form-item label="状态">
        <el-select v-model="searchForm.state">
          <el-option
            v-for="item in status"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button @click="clear">清除</el-button>
        <el-button type="primary" @click="search">搜索</el-button>
      </el-form-item>
    </el-form>
    <!-- 表格 -->
    <div>
      <el-table :data="list">
        <el-table-column prop="id" label="序号"></el-table-column>
        <el-table-column prop="directoryName" label="目录名称"></el-table-column>
        <el-table-column prop="creator" label="创建者"></el-table-column>
        <el-table-column prop="adddate" label="创建时间"></el-table-column>
        <el-table-column prop="totals" label="面试题数量"></el-table-column>
        <el-table-column :formatter="formatterBool" prop="state" label="状态"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button type="text" @click="modify(scope.row.id)">修改</el-button>
            <el-button @click="forbidden(scope.row)" type="text">{{scope.row.state===1?'禁用':'启用'}}</el-button>
            <!-- <el-button type="text" @click="deldir(scope.row.id)">删除</el-button> -->
            <el-button @click="deldir({'id':scope.row.id})" type="text">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <el-row type="flex" justify="end">
        <el-pagination
          background
          layout="prev, pager, next"
          :current-page="page.currentPage"
          :page-size="page.pageSize"
          :total="page.total"
        ></el-pagination>
      </el-row>
    </div>
    <!-- 放置一个弹层 -->
    <el-dialog :show-close="false" :visible="dialogVisible" @close="beforeClose">
      <el-form ref="myForm" :model="formData" :rules="rules" label-width="180px">
        <el-form-item label="目录名称" prop="directoryName">
          <el-input v-model="formData.directoryName"></el-input>
        </el-form-item>
        <el-form-item label="学科" prop="subjectID">
          <el-select v-model="formData.subjectID">
            <el-option
              v-for="item in option"
              :key="item.value"
              :value="item.value"
              :label="item.label"
            ></el-option>
          </el-select>
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
import Vue from "vue";
import directorysadd from "../components/directorys-add";
import { list as directoryslist } from "../../api/hmmm/directorys";
import { add } from "../../api/hmmm/directorys";
import { removeState } from "../../api/hmmm/directorys";
import { status } from "../../api/hmmm/constants";
import { remove } from "../../api/hmmm/directorys";
import { detail } from "../../api/hmmm/directorys";
import { simple as subjects } from "../../api/hmmm/subjects";
import { update } from "../../api/hmmm/directorys";

Vue.component("directorysadd", directorysadd);
export default {
  data() {
    return {
      dialogVisible: false,
      formData: {
        directoryName: "",
        subjectID: ""
      },
      option: [],
      rules: {
        directoryName: [
          { required: true, message: "目录名称不能为空", trigger: "blur" }
        ],
        subjectID: [
          { required: true, message: "学科不能为空", trigger: "blur" }
        ]
      },
      searchForm: {
        directoryName: "",
        state: ""
      },
      list: [],
      data: {},
      status,
      page: {
        currentPage: 1,
        pageSize: 10,
        total: 0
      }
    };
  },
  methods: {
    resetFields() {
      // this.$refs.myForm.resetFields();
      this.formData = {
        directoryName: "",
        subjectID: ""
      };
      this.$refs.myForm.clearValidate();
    },
    beforeClose() {
      this.resetFields();
      this.dialogVisible = false;
    },
    async btnOK() {
      await this.$refs.myForm.validate();
      // console.log(this.formData)
       await update(this.formData) 
      //  console.log(1);
      this.$message({
        type: "success",
        message: "操作成功"
      });
      this.resetFields();
      this.getDirectory();
      this.dialogVisible = false;
    },
    // 学科简单列表
    simple() {
      subjects().then(result => {
        this.option = result.data;
        console.log(result.data);
      });
    },
    // 修改
    async modify(id) {
      let result = await detail({ id });
      this.formData = result.data;
      this.dialogVisible = true;
    },
    // 删除目录
    async deldir(id) {
      await this.$confirm("确认删除此目录吗");
      await remove(id);
      this.$message({ type: "success", message: "删除成功" });
      this.getlist();
    },
    // 输入框为空时，重新加载数据
    blur() {
      this.getlist();
    },
    // 搜索
    search() {
      console.log(111);
      this.page.currentPage = 1;
      this.getDirectory(this.searchForm);
    },
    // 清除内容
    clear() {
      this.searchForm = {
        directoryName: "",
        state: null
      };
      this.getlist();
    },
    // 获取目录数据
    async getDirectory(data) {
      let result = await directoryslist({
        page: this.page.currentPage,
        pageSize: this.page.pageSize,
        ...data //合并条件
      });
      this.page.total = result.data.counts;
      this.list = result.data.items;
    },
    async forbidden(data) {
      // console.log(data);
      await this.$confirm("确认改变状态吗");
      await removeState({ id: data.id, state: data.state == 1 ? 0 : 1 });
      this.getlist();
    },
    diradd(data) {
      console.log();

      add(data).then(result => {
        // alert("11111111111111");
        this.getlist();
      });
    },
    // 新增目录
    formatterBool(row, column, cellValue) {
      return cellValue ? "正常" : "关闭";
    },
    getlist() {
      directoryslist().then(result => {
        this.list = result.data.items;
      });
    }
  },
  created() {
    this.getDirectory();
    this.getlist();
    this.simple();
  }
};
</script>

<style scoped>
</style>
