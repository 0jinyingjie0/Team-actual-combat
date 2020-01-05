<template>
  <el-card>
    <el-row slot="header">
      <el-button type="primary" @click="addtest">新增试题</el-button>
      <el-button type="primary">批量导入</el-button>
    </el-row>
    <el-form inline ref="myForm" :model="formData" label-width="80px">
      <el-form-item label="学科" prop="subjectID">
        <el-select v-model="formData.subjectID" placeholder="请选择">
          <el-option
            v-for="item in subjects"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="难度">
        <el-select v-model="formData.difficulty">
          <el-option
            v-for="item in difficulty"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="试题类型">
        <el-select v-model="formData.type">
          <el-option v-for="item in type" :key="item.value" :label="item.label" :value="item.value"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="标签">
        <el-select v-model="formData.tags">
          <el-option v-for="item in tags" :key="item.value" :label="item.label" :value="item.value"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="城市">
        <el-select v-model="formData.province">
          <el-option v-for="(item,index) in provinces" :key="index" :label="item" :value="item"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="区域">
        <el-select v-model="formData.citys">
          <el-option v-for="(item,index) in getcitys" :key="index" :label="item" :value="item"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="关键字">
        <el-input v-model="formData.keyword"></el-input>
      </el-form-item>
      <el-form-item label="题目备注">
        <el-input v-model="formData.title"></el-input>
      </el-form-item>
      <el-form-item label="企业简称">
        <el-input v-model="formData.enterprise"></el-input>
      </el-form-item>
      <el-form-item label="方向">
        <el-select v-model="formData.direction">
          <el-option v-for="(item,index) in directions" :key="index" :label="item" :value="index"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="录入人">
        <el-select v-model="formData.creator">
          <el-option
            v-for="item in creator"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="二级目录">
        <el-select v-model="formData.secondlist">
          <el-option
            v-for="item in secondlists"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>
      </el-form-item>
      <!-- {{this.formData.province}} -->
      <div style="width:100%;display:flex;justify-content:center;margin-top:20px">
        <el-form-item>
          <el-button @click="clear">清除</el-button>
          <el-button type="primary" @click="search">搜索</el-button>
        </el-form-item>
      </div>
    </el-form>
    <el-table style="width:100%" :data="list">
      <el-table-column label="序号" prop="id"></el-table-column>
      <el-table-column label="试题编号" prop="number"></el-table-column>
      <el-table-column label="学科" prop="subjectID"></el-table-column>
      <el-table-column :formatter="formatterque" label="题型" prop="questionType"></el-table-column>
      <el-table-column label="题干" width="200" prop="question"></el-table-column>
      <el-table-column label="录入时间" width="200" prop="addDate"></el-table-column>
      <el-table-column :formatter="formatterdif" label="难度" prop="difficulty"></el-table-column>
      <el-table-column label="录入人" prop="creator"></el-table-column>
      <el-table-column label="操作" width="200">
        <template slot-scope="obj">
          <el-button type="text">预览</el-button>
          <el-button type="text">修改</el-button>
          <el-button type="text" @click="delItem(obj.row.id)">删除</el-button>
          <el-button type="text" @click="joinItem(obj.row.id)">加入精选</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
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
  </el-card>
</template>

<script>
import { list as questions } from "../../api/hmmm/questions";
import { simple as subjects } from "../../api/hmmm/subjects";
import { simple as tags } from "../../api/hmmm/tags";
import { simple as directorys } from "../../api/hmmm/directorys";
import { direction } from "../../api/hmmm/constants";
import { questionType } from "../../api/hmmm/constants";
import { difficulty } from "../../api/hmmm/constants";
import { provinces, citys } from "../../api/hmmm/citys";
import { remove } from "../../api/hmmm/questions";
export default {
  data() {
    return {
      page: {
        currentPage: 1,
        pageSize: 10,
        total: 0
      },
      list: [],
      secondlists: [],
      // 方向
      directions: direction,
      // 区域
      // arealist: [],
      // 城市
      provinces: provinces(),
      // 标签
      tags: [],
      // 学科
      subjects: [],
      value: "",
      // 难度
      difficulty: difficulty,
      // 试题类型
      type: questionType,
      creator: [
        {
          value: "1",
          label: "超级管理员"
        },
        {
          value: "2",
          label: "编辑"
        }
      ],
      formData: {
        subjectID: "", //学科
        difficulty: "", //难度
        type: "", //类型
        tags: "", //标签
        province: "", //城市
        citys: "", //地区
        keyword: "", //关键字
        title: "", //题目备注
        enterprise: "", //企业简称
        direction: "", //方向
        creator: "", //录入人
        secondlist: "" //二级目录
      }
    };
  },
  methods: {
    async joinItem(id) {
      await this.$confirm("是否加入精选");
    },
     // 删除文章
    async delItem(id) {
      await this.$confirm("是否删除此文章");
      await remove({ id });
      this.$message({ type: "success", message: "删除成功" });
      this.getquestionlist();
    },
      changePage(newPage) {
      this.page.currentPage = newPage;
      this.getquestionlist();
    },
    addtest() {
      this.$router.push("new");
    },
    formatterdif(row, column, cellValue) {
      if (cellValue == 1) {
        return "简单";
      } else if (cellValue == 2) {
        return "一般";
      } else if (cellValue == 3) {
        return "困难";
      } else {
        return "未知";
      }
    },
    formatterque(row, column, cellValue) {
      if (cellValue == 1) {
        return "单选";
      } else if (cellValue == 2) {
        return "多选";
      } else if (cellValue == 3) {
        return "简答";
      } else {
        return "未知";
      }
    },
    clear() {
      this.formData.subjects = "";
      this.formData.difficulty = "";
      this.formData.type = "";
      this.formData.tags = "";
      this.formData.province = "";
      this.formData.citys = "";
      this.formData.keyword = "";
      this.formData.title = "";
      this.formData.enterprise = "";
      this.formData.direction = "";
      this.formData.creator = "";
      this.formData.secondlist = "";
    },
    search() {
      this.page.currentPage = 1;
      this.getquestionlist();
    },
    // 基础题库类表
    getquestionlist() {
      let params = {
        page: this.page.currentPage,
        pageSize: this.page.pageSize,
        ...this.formData
      };
      questions(params).then(result => {
        // console.log(result.data);
        this.list = result.data.items;
        this.page.total = result.data.counts;
      });
    },
    // 学科下拉
    subjectslist() {
      subjects().then(result => {
        this.subjects = result.data;
        // console.log(this.subjects);
      });
    },
    // 标签下拉
    tagslist() {
      tags().then(result => {
        this.tags = result.data;
      });
    },
    // 目录下啦
    directory() {
      directorys().then(result => {
        this.secondlists = result.data;
      });
    },
    // 城市下拉
    provincelist() {
      // console.log(provinces());
      this.provinces = provinces();
      // console.log(this.provinces);
    }
  },
  // 计算属性，获取城市下地区
  computed: {
    getcitys() {
      return citys(this.formData.province);
    }
  },
  created() {
    this.getquestionlist();
    this.subjectslist();
    this.tagslist();
    this.directory();
    this.provincelist();
  }
};
</script>

<style lang="scss">
</style>
