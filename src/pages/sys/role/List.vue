<template>
  <!-- 角色管理 -->
  <div class="role_list">
    <div class="btns">
      <el-button type="primary" size="small" @click="toAdd">添加</el-button>
    </div>
    <el-table :data="roles" size="small">
      <el-table-column prop="name" label="角色名称" />
      <el-table-column label="操作" align="center" width="180">
        <template slot-scope="scope">
          <a @click.prevent="deleteHandler(scope.row.id)">移除</a>
          <a @click.prevent="toAuthorization(scope.row)">授权</a>
        </template>
      </el-table-column>
    </el-table>
    <!-- 模态框 -->
    <el-dialog :title="title" :visible.sync="visible">
      <el-form :model="form">
        <el-form-item label="角色名称" label-width="80px">
          <el-input v-model="form.name" autocomplete="off" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button size="small" @click="visible = false">取 消</el-button>
        <el-button type="primary" size="small" @click="saveRoleHandler">确 定</el-button>
      </div>
    </el-dialog>
    <!-- /模态框 -->
    <!-- 授权模态框 -->
    <el-dialog title="授权" :visible.sync="authorization_visible" width="80%">
      <el-form :model="role">
        <!-- {{role}} -->
        <el-form-item label="角色名称" label-width="80px">
          {{ role.name }}
        </el-form-item>
        <el-form-item label="权限" label-width="80px">
          <!-- <el-cascader-panel v-model="role.privileges" :options="options" :props="props" clearable></el-cascader-panel> -->
          <el-cascader-panel v-model="role.privileges" :options="options" :props="props" clearable />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button size="small" @click="authorization_visible = false">取 消</el-button>
        <el-button type="primary" size="small" @click="authorizationHandler">确 定</el-button>
      </div>
    </el-dialog>
    <!-- /模态框 -->
  </div>
</template>
<script>
import request from '@/utils/request'
import qs from 'querystring'
export default {
  data() {
    return {
      form: {},
      visible: false,
      authorization_visible: false,
      title: '添加角色',
      role: {}, // 当前角色
      roles: [], // 角色列表
      props: { multiple: true, value: 'id', label: 'name', emitPath: false, checkStrictly: true },
      options: []
    }
  },
  created() {
    // 加载角色
    this.loadRoles()
    // 加载权限
    this.loadPrivileges()
  },
  methods: {
    authorizationHandler() {
      // let privileges =[];
      // for(let item of this.role.privileges){
      //   privileges.push(...item)
      // }
      // privileges = [...new Set(privileges)]
      // let param = {
      //   id:this.role.id,
      //   privileges
      // }
      request.request({
        url: '/role/authorization',
        method: 'post',
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        },
        data: qs.stringify(this.role)
      })
        .then(response => {
          this.authorization_visible = false
          this.$message({ message: response.message, type: 'success' })
          this.loadRoles()
        })
    },
    saveRoleHandler() {
      request.request({
        url: '/role/saveOrUpdate',
        method: 'post',
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        },
        data: qs.stringify(this.form)
      })
        .then(response => {
          this.visible = false
          this.$message({ message: response.message, type: 'success' })
          this.loadRoles()
        })
    },
    loadPrivileges() {
      request.get('/privilege/findPrivilegeTree')
        .then(response => {
          this.foo(response.data)
          this.options = response.data
        })
    },
    // 递归去除权限中的空children
    foo(privileges) {
      for (const p of privileges) {
        if (p.children && p.children.length === 0) {
          delete p.children
        } else {
          this.foo(p.children)
        }
      }
    },
    toAdd() {
      this.visible = true
    },
    loadRoles() {
      request.get('/role/cascadePrivilegeFindAll')
        .then(response => {
        // 将权限转换为id的数组
          response.data.forEach(item => {
            item.privileges = item.privileges.map(p => {
              return p.id
            })
          })
          this.roles = response.data
        })
    },
    deleteHandler(id) {
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        request.get('/role/deleteById?id=' + id)
          .then(response => {
            this.$message({ type: 'success', message: response.message })
            this.loadRoles()
          })
      })
    },
    toAuthorization(record) {
      this.role = record
      this.authorization_visible = true
    }
  }
}
</script>
