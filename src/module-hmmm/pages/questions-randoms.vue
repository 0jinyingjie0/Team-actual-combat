<template>
  <!-- 组题列表 -->
  <el-card class="dashboard-container" style="margin:5px 10px">
    <!-- 表格 -->
    <el-table :data="items">
        <!-- 列 -->
      <el-table-column label="序号" prop="id"></el-table-column>
      <el-table-column label="用户名" prop="userName"></el-table-column>
      <el-table-column label="试题ID" prop="questionIDs"></el-table-column>
      <el-table-column label="答题进度" prop="progressOfAnswer"></el-table-column>
      <el-table-column label="正确率" prop="accuracyRate"></el-table-column>
      <el-table-column label="答题总耗时" prop="totalSeconds"></el-table-column>
      <el-table-column label="组题类型/详情" prop="questionType"></el-table-column>
     
          <el-table-column label="操作">
             <template slot-scope="obj">
           <el-button @click="delData(obj.row.id)" type="text">删除</el-button>
             </template>
            </el-table-column>    
    </el-table> 
     <!-- 分页组件 -->
      <el-row type="flex" justify="center" align="middle" style="height:80px">
        <el-pagination background layout="prev, pager, next"
         :total="page.total"
         :page-size="page.pageSize"
         :current-page="page.currentPage"
         @current-change="changePage"
         >
         </el-pagination>
      </el-row>
  </el-card>
</template>

<script>
import { randoms ,removeRandoms} from "../../api/hmmm/questions";
export default {
  data() {
    return {
      // 接收列表数据
      items: [],
      page: {
        counts:0,  //总记录数
        total: 0, // 总页数
        pageSize: 10, // 每页显示条目个数
        currentPage: 1// 当前页数，
      }
    };
  },
  methods: {
    // 定义参数 条件
    getCondition(){
       let params = {
        page: this.page.currentPage,
        pageSize: this.page.pageSize
      };
      this.getData(params)
    },
    // 获取数据方法
    getData(data) {
      randoms(data).then(result => {
        this.items = result.data.items;
        this.page.total = result.data.pages; //获取总页数
        
        
        //  console.log( result.data.pages);
      });
    },
    // 改变当前页码
   changePage (newPage) {
      this.page.currentPage = newPage // 改变当前页
      this.getCondition()
    },
    // 删除
    delData(id) {
      this.$confirm("您确定要删除吗？").then(() => {
        // alert(data.id)
        //  调接口
        removeRandoms({id}).then(() => {
          this.$message({
            type: "success",
            message: "删除成功！"
          });
          // 重新调用
          this.getCondition();
        });
      });
    }
  },
  created() {
    this.getData();
  }
};
</script>

<style scoped>
</style>
