<template>
  <div class="article_list">
    <!-- 按钮 -->
    <div class="btns">
      <el-button type="primary" size="small" @click="toPublishArticle">发布文章</el-button>
    </div>
    <!-- 表格 -->
    <el-table :data="articles" style="width: 100%">
      <el-table-column prop="id" label="编号" width="180">
      </el-table-column>
      <el-table-column prop="title" label="标题" width="180">
      </el-table-column>
      <el-table-column prop="author.username" label="作者">
      </el-table-column>
      <el-table-column prop="category.name" label="所属栏目">
      </el-table-column>
      
      <el-table-column label="操作" align="center">
        <template slot-scope="scope">
          <el-button size="mini" >查看</el-button>
          <el-button size="mini" type="primary" @click="handleEdit(scope.row)">编辑</el-button>
          <el-button size="mini" type="danger" @click="handleDelete(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>

    </el-table>
    <!-- 分页 -->
  </div>
</template>
<script>
import request from '@/utils/request'
import qs from 'querystring'
export default {
  //为模板提供数据源泉
  data(){
    return {
      articles:[],
    }
  },
  //生命周期
  created(){
    //调用ajax查询数据
    request.get("/article/findAll")
    .then(result=>{
      this.articles = result.data;
    })
  },
  //事件处理函数
  methods:{
    // 重载数据
    reload(){
      request.get("/article/findAll")
      .then(result=>{
      this.articles = result.data;
      })
    },

    //发布博客
    toPublishArticle(){
      // 跳转到编辑页面
      this.$router.push({path:'/article/editor'})
    },
   

    // 删除文章
    handleDelete(id){
      this.$confirm('永久删除该博客'+ id +', 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          let url ="/article/dropOne"
          request.get(url,{params:{id}})
          // 响应通知
          .then(response =>{
            this.$message({
              message:response.message,
              type:"success"
            })
            this.reload();
          })
        })
    },

    // 修改文章
    handleEdit(edit) {
      this.$router.push({path:'/article/editor',query:edit})
    }
  }
}
</script>
<style scoped>

</style>