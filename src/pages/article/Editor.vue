<template>
  <div class="article_editor" style="alien:center">

    {{form}}
    <el-form ref="form" :model="form" label-width="80px">

      <el-form-item label="作  者">
        <el-select v-model="form.authorId" placeholder="博客作者" @click.native="userHandler()">
          <el-option v-for="user in users" :key="user.id" :value="user.id" :label="user.username"></el-option>
        </el-select>
      </el-form-item>
      
      <el-form-item label="所属栏目">
        <el-select v-model="form.categoryId" placeholder="博客栏目" @click.native="categoryHandler()">
          <el-option v-for="cate in categorys" :key="cate.id" :value="cate.id" :label="cate.name"></el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="博客标题">
        <el-input v-model="form.title"></el-input>
      </el-form-item>

      <el-form-item label="正  文">
        <el-input type="textarea" v-model="form.content" rows="10"></el-input>
      </el-form-item>

      <div >
        <el-form-item>
          <el-button type="primary" @click="onSubmit">提交</el-button>
          <el-button style="margin-left: 40px" type="primary" @click="back">返回</el-button>
        </el-form-item>
      </div>
    </el-form>
    

  </div>
</template>
<script>
import request from '@/utils/request'
import qs from 'querystring'
export default {
  data(){
    return {
      form:{},
      categorys:{},
      users:{}
    }
  },
  created() {
    this.form = this.$route.query;
  },
  methods:{

    back(){
      this.$router.go(-1);
    },

    onSubmit(){
      request({
        method:"post",
        url:"/article/saveOrUpdate",
        //将json数据转化为字符转拼接，存储到请求体中通过post，put，patch方法提交
        data:qs.stringify(this.form),
        headers:{
          //告诉后台请求体数据编码格式，数据类型
          'Content-Type':'application/x-www-form-urlencoded'
        }
      })
      .then(result=>{
        this.$message({
          message: '恭喜你，这是一条成功消息',
          type: 'success'
        });
      });
      this.back();
    },

    // 栏目信息下拉框查询
    categoryHandler(){
      request.get("/category/findAll")
      .then(result=>{
        this.categorys = result.data;
      })
    },

    // 栏目信息下拉框查询
    userHandler(){
      request.get("/user/findAll")
      .then(result=>{
        this.users = result.data;
      })
    }
  }
}
</script>
