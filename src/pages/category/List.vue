<template>
    <div class="category_list" style="align:center">
    {{tableData}}
        <el-table ref="multipleTable" :data="tableData" tooltip-effect="dark" style="width: 100%" @selection-change="handleSelectionChange">
            <el-table-column type="selection" width="55"> </el-table-column>
            <el-table-column prop="id" label="栏目ID" width="120"></el-table-column>
            <el-table-column prop="name" label="栏目名称" width="120"> </el-table-column>
            <el-table-column prop="description" label="栏目描述" width="120"> </el-table-column>
            <el-table-column prop="parentId" label="父栏目ID" show-overflow-tooltip> </el-table-column>
            <el-table-column label="操作" align="center">
            <template slot-scope="scope">
            <el-button size="mini" type="primary" @click="handleEdit(scope.row)">编辑</el-button>
            <el-button size="mini" type="danger" @click.native="handleDelete(scope.row.id)">删除</el-button>
            </template>
            </el-table-column>
        </el-table>

        <div style="margin-top: 20px">
            <el-button @click="dialogFormVisible = true">添加栏目</el-button>
            <el-button @click="toggleSelection(tableData)">切换全部选中状态</el-button>
            <el-button type="primary" @click="toggleSelection()">取消已选择栏目</el-button>
            <el-button type="danger" @click="deleteSelection(ids)">批量删除栏目</el-button>
        </div>

        <!-- Dialog -->
        <el-dialog title="栏目信息" :visible.sync="dialogFormVisible">
    {{dialog}}
        <el-form :model="dialog">
            <el-form-item label="父栏目名称" :label-width="formLabelWidth">
                <el-select v-model="dialog.parentId" placeholder="父栏目" clearable>
                    <el-option v-for="cate in tableData" :key="cate.id" :value="cate.id" :label="cate.name">
                    </el-option>
                </el-select>
            </el-form-item>
            <el-form-item label="栏目名称" :label-width="formLabelWidth">
                <el-input v-model="dialog.name" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="栏目描述" :label-width="formLabelWidth">
                <el-input type="textarea" v-model="dialog.description" autocomplete="off"></el-input>
            </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
            <el-button type="primary" @click="handleSubmit()">添 加</el-button>
            <el-button @click="dialogFormVisible = false">取消添加</el-button>
        </div>
        </el-dialog>
        <!-- /Dialog -->
    </div>
</template>

<script>
import request from '@/utils/request'
import qs from 'querystring'
// 数据源
  export default {
    data() {
      return {
        tableData: [],
        ids:[],
        dialogFormVisible: false,
        formLabelWidth: '120px',
        dialog: {},
        null:{}
      }
    },

// 生命周期
    created(){
        request.get("http://127.0.0.1:8989/category/findAll")
        .then(result=>{
            this.tableData=result.data
        })
    },

// 方法
    methods: {

    // 重新加载数据
        reload() {
             request.get("http://127.0.0.1:8989/category/findAll")
            .then(result=>{
                this.tableData=result.data
            })
        },

    // 选择或者取消
        toggleSelection(rows) {
            if (rows) {
                rows.forEach(row => {
                this.$refs.multipleTable.toggleRowSelection(row);
                });
            } else {
                this.$refs.multipleTable.clearSelection();
            }
            },

    //选择单行栏目
        handleSelectionChange(val) {
            this.ids = val.map(item=>item.id);
        },

    // 删除单个栏目
        handleDelete(id) {
            this.$confirm('删除该'+ id +'号栏目, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
            }).then(() => {
            let url ="http://localhost:8989/category/dropOne"
            request.get(url,{params:{id}})
            // 响应通知
            .then(response =>{
                this.$message({
                message:response.message,
                type:"success"
                });
                this.reload();
            })
            })
        },

    //  删除多个栏目
        deleteSelection(ids) { 
            this.$confirm('删除该栏目, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
            }).then(() => {
            let url ="http://localhost:8989/category/dropSome"
            request.post(url,ids)
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
    
    // 添加栏目
        handleSubmit(){
            request({
                method:"post",
                url:"http://127.0.0.1:8989/category/saveOrUpdate",
                //将json数据转化为字符转拼接，存储到请求体中通过post，put，patch方法提交
                data:qs.stringify(this.dialog),
                headers:{
                    //告诉后台请求体数据编码格式，数据类型
                'Content-Type':'application/x-www-form-urlencoded'
                }
            })
            .then(result=>{
                this.dialogFormVisible = false
                this.reload()
                this.$message({
                    message: '恭喜你，这是一条成功消息',
                type: 'success'
                });
            })
        },

    //添加栏目
        handleAdd(){
            this.dialog=this.null
            this.dialogFormVisible = true
        },

    // 编辑栏目
        handleEdit(row){
            this.dialog=row
            this.dialogFormVisible = true
        }

    }

  }
</script>