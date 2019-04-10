<template>
  <div class="roles">
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 增加按钮 -->
    <el-button type="success" plain @click="showAddDialog">增加角色</el-button>
    <!-- 表格组件 -->
    <el-table :data="roleList">
      <el-table-column type="expand">
        <template slot-scope="{row}">
          <span v-if="row.children.length === 0">暂无权限</span>
          <!-- 显示一级分类 -->
          <el-row class="l1" v-for="l1 in row.children" :key="l1.id">
            <el-col :span="4">
              <el-tag @close="delRight(row, l1.id)" closable>{{l1.authName}}</el-tag>
              <i class="el-icon-arrow-right"></i>
            </el-col>
            <el-col :span="20">
              <!-- 显示二级分类 -->
              <el-row class="l2" v-for="l2 in l1.children" :key="l2.id">
                <el-col :span="4">
                  <el-tag @close="delRight(row, l2.id)" closable type="success">{{l2.authName}}</el-tag>
                  <i class="el-icon-arrow-right"></i>
                </el-col>
                <el-col :span="20">
                  <!-- 显示3级分类 -->
                  <span v-for="l3 in l2.children" :key="l3.id">
                    <el-tag
                      @close="delRight(row, l3.id)"
                      closable
                      class="l3"
                      type="warning"
                    >{{l3.authName}}</el-tag>
                  </span>
                </el-col>
              </el-row>
            </el-col>
          </el-row>
        </template>
      </el-table-column>
      <el-table-column type="index"></el-table-column>
      <el-table-column label="角色名称" prop="roleName"></el-table-column>
      <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
      <el-table-column label="操作">
        <template slot-scope="{row}">
          <el-button type="primary" plain icon="el-icon-edit" size="mini" @click="showEditDialog(row)"></el-button>
          <el-button type="danger" plain icon="el-icon-delete" size="mini" @click="delRole(row.id)"></el-button>
          <el-button
            type="success"
            plain
            icon="el-icon-check"
            @click="showAssignDialog(row)"
            size="mini"
          >分配权限</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分配角色对话框 -->
    <el-dialog title="角色授权" :visible.sync="assignDialogVisible" width="40%">
      <!-- 树形菜单 -->
      <!--
        :data: 树形菜单的数据
        props：用于指定显示的属性以及设置子属性
        show-checkbox：显示checkbox
        default-expand-all: 默认展开所有节点
      -->
      <el-tree
        ref="tree"
        :data="data"
        node-key="id"
        :props="defaultProps"
        show-checkbox
        default-expand-all
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="assignDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="assignRight">分 配</el-button>
      </span>
    </el-dialog>

    <el-dialog :title="this.addForm.id ? '修改角色' : '添加角色'" :visible.sync="addDialogVisible" width="40%">
      <el-form status-icon ref="addForm" :rules="rules" :model="addForm" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addForm.roleName" placeholder="请输入角色名称"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addForm.roleDesc" placeholder="请输入角色描述"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveRole">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      roleList: [],
      assignDialogVisible: false,
      data: [],
      defaultProps: {
        children: 'children',
        label: 'authName'
      },
      roleId: '',
      addDialogVisible: false,
      addForm: {
        id: '',
        roleName: '',
        roleDesc: ''
      },
      rules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    // 获取角色列表数据
    async getRoleList() {
      let res = await this.axios.get('roles')
      if (res.meta.status === 200) {
        this.roleList = res.data
        console.log(this.roleList)
      }
    },
    // 知识点：记忆性   理解性   //递归
    async delRight(role, rightId) {
      let res = await this.axios.delete(`roles/${role.id}/rights/${rightId}`)
      let {
        meta: { status },
        data
      } = res
      if (status === 200) {
        // this.getRoleList()
        role.children = data
        this.$message.success('取消权限成功了')
      } else {
        this.$message.info('取消权限失败')
      }
    },
    async showAssignDialog(role) {
      // 0. 存储了需要分配的角色id
      this.roleId = role.id
      // 1.显示对话框
      this.assignDialogVisible = true
      // 2. 发送ajax请求，获取到所有的权限的数据， 树状
      let res = await this.axios.get('rights/tree')
      let {
        meta: { status },
        data
      } = res
      if (status === 200) {
        this.data = data
        console.log(this.data)
      }
      // 3. 使用树形菜单把数据显示出来
      // 4. 默认选中， 如何让节点默认选中   让谁选中（当前角色所有的3级权限）
      // 找到当前角色有拥有的所有的三级权限
      // console.log(role)
      let ids = []
      role.children.forEach(l1 => {
        l1.children.forEach(l2 => {
          l2.children.forEach(l3 => {
            ids.push(l3.id)
          })
        })
      })
      this.$refs.tree.setCheckedKeys(ids)
    },
    async assignRight() {
      // 获取到所有选中的id
      let checkedKeys = this.$refs.tree.getCheckedKeys()
      let halfCheckedKeys = this.$refs.tree.getHalfCheckedKeys()
      let rids = checkedKeys.concat(halfCheckedKeys).join()
      // 发送ajax请求
      let res = await this.axios.post(`roles/${this.roleId}/rights`, {
        rids
      })
      let {
        meta: { status }
      } = res
      if (status === 200) {
        this.getRoleList()
        this.assignDialogVisible = false
        this.$message.success('分配权限成功了')
      } else {
        this.$message.error('分配权限失败了')
      }
    },
    showAddDialog() {
      this.addDialogVisible = true
      this.addForm.roleName = ''
      this.addForm.roleDesc = ''
      this.addForm.id = ''
    },
    saveRole() {
      // 1. 表单校验
      this.$refs.addForm.validate(async valid => {
        if (!valid) return false

        // 2.发送ajax请求
        // 请求方式 post  put
        // 请求地址 'roles'     'roles/id'
        // 响应结果： 201       200
        let id = this.addForm.id
        let method = id ? 'put' : 'post'
        let url = id ? `roles/${id}` : `roles`
        let status = id ? 200 : 201
        let res = await this.axios({
          method,
          url,
          data: this.addForm
        })
        if (res.meta.status === status) {
          this.$refs.addForm.resetFields()
          this.addDialogVisible = false
          this.getRoleList()
          // 提示消息
          this.$message.success('操作成功')
        } else {
          this.$message.error('操作失败')
        }
      })
    },
    showEditDialog(role) {
      this.addDialogVisible = true
      // 回显数据
      this.addForm.roleName = role.roleName
      this.addForm.roleDesc = role.roleDesc
      this.addForm.id = role.id
    },
    async delRole(id) {
      try {
        await this.$confirm('你确定要删除该角色吗?', '温馨提示', {
          type: 'warning'
        })
        // 发送ajax请求
        let res = await this.axios.delete(`roles/${id}`)
        if (res.meta.status === 200) {
          // 删除成功
          this.getRoleList()
          this.$message.success('删除成功')
        }
      } catch (e) {
        this.$message.info('取消删除')
      }
    }
  },
  created() {
    this.getRoleList()
  }
}
</script>

<style lang="less" scoped>
.l1 {
  border-bottom: 1px dotted #ccc;
  padding: 10px 0;
}

.l2 {
  margin-bottom: 5px;
}

.l3 {
  margin-right: 5px;
  margin-bottom: 5px;
}
</style>
