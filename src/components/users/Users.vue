<template>
  <div class="users">
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>活动列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索栏 -->
    <el-input placeholder="请输入内容" v-model="query" class="input-with-select">
      <el-button slot="append" icon="el-icon-search" @click="search"></el-button>
    </el-input>
    <el-button type="success" plain @click="showAddDialog">添加用户</el-button>
    <!-- 表格组件 -->
    <!--
      el-table : 表格组件
      data : 指定表格最终需要渲染的数据（数组）
      style： 让表格宽度100%
      el-table-column： 定义表格的每一列
        label="日期"： 列的标题
        prop： 对应显示的数据的属性名
        width： 列的宽度 不支持百分比

      在el-tabel中，如果想要自定义列模版
      在el-table-column中使用template
        <template slot-scope="scope">自己定义的内容</template>
    -->
    <el-table :data="userList" style="width: 100%">
      <el-table-column prop="username" label="姓名" width="180"></el-table-column>
      <el-table-column prop="email" label="邮箱" width="180"></el-table-column>
      <el-table-column prop="mobile" label="电话"></el-table-column>
      <el-table-column prop="mg_state" label="用户状态">
        <!-- 在自定义列模版中，如何访问到当前列的数据 -->
        <template slot-scope="scope">
          <el-switch
            @change="changeState(scope.row)"
            v-model="scope.row.mg_state"
            active-color="#13ce66"
            inactive-color="#ff4949"
          ></el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button
            type="primary"
            icon="el-icon-edit"
            plain
            size="mini"
            @click="showEditDialog(scope.row)"
          ></el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            @click="delUser(scope.row.id)"
            plain
            size="mini"
          ></el-button>
          <el-button @click="showAssignDialog(scope.row)" type="success" icon="el-icon-check" plain size="mini">分配角色</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!--
      分页
      @size-change: 表示每页的条数发生了改变，会触发handleSizeChange
      @current-change: 当前页发生改变
      current-page: 指定当前页面
      page-sizes: 指定选择每页条数的数组
      page-size: 每页的条数
      total:指定总条数
      layout: 指定分页的空间
    -->
    <!--
      1. 找到分页组件
       <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage4"
        :page-size="100"
        :total="400"
        :page-sizes="[100, 200, 300, 400]"
        layout="total, sizes, prev, pager, next, jumper"
       >
      </el-pagination>
      1. size-change: 修改了每页的条数
      2. current-change： 修改了页码
      3. 指定  pageSize  currentPage  total: 获取用户列表的时候
      4. page-sizes: 每页的条数   pageSize
      5.  layout： 控制分页控制的显示的顺序

    -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage"
      :page-sizes="[2, 4, 6, 8]"
      :page-size="pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
      background
    ></el-pagination>

    <!-- 添加用户的对话框 -->
    <!--
      el-dialog：整个对话框组件
      visible： 对话框是否可见
    -->
    <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="40%">
      <!-- 添加表单 -->
      <el-form ref="addForm" :model="addForm" label-width="80px" :rules="rules" status-icon>
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username" placeholder="请输入用户名"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password" placeholder="请输入密码"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email" placeholder="请输入用户邮箱"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="addForm.mobile" placeholder="请输入用户手机"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="40%">
      <!-- 修改表单 -->
      <el-form ref="editForm" :model="editForm" label-width="80px" :rules="rules" status-icon>
        <el-form-item label="用户名">
          <el-tag type="info">{{editForm.username}}</el-tag>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email" placeholder="请输入用户邮箱"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="editForm.mobile" placeholder="请输入用户手机"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="updateUser">确 定</el-button>
      </span>
    </el-dialog>

    <el-dialog title="分配角色" :visible.sync="assignDialogVisible" width="40%">
       <el-form ref="assignForm" :model="assignForm" label-width="80px" :rules="rules" status-icon>
        <el-form-item label="用户名">
          <el-tag type="info">{{assignForm.username}}</el-tag>
        </el-form-item>
        <el-form-item label="角色列表">
            <el-select v-model="assignForm.rid" placeholder="请选择">
              <!-- options:需要是角色的列表  -->
              <el-option
                v-for="item in roleList"
                :key="item.id"
                :label="item.roleName"
                :value="item.id">
              </el-option>
            </el-select>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="assignDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="assignRole">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
// 引入axios
// import axios from 'axios'
export default {
  data() {
    return {
      // 用户列表
      userList: [],
      query: '',
      currentPage: 1,
      pageSize: 2,
      total: 0,
      // 控制添加用户的对话框的显示与隐藏
      addDialogVisible: false,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      rules: {
        username: [
          { required: true, message: '用户名不能为空', trigger: 'blur' },
          { min: 3, max: 9, message: '用户名长度在3-9位', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '密码不能为空', trigger: 'blur' },
          { min: 6, max: 12, message: '密码长度在6-12位', trigger: 'blur' }
        ],
        email: [
          { type: 'email', message: '请输入一个合法的邮箱', trigger: 'blur' }
        ],
        mobile: [
          {
            pattern: /^1\d{10}$/,
            message: '请输入一个合法的手机号',
            trigger: 'blur'
          }
        ]
      },
      editDialogVisible: false,
      editForm: {
        id: '',
        username: '',
        email: '',
        mobile: ''
      },
      assignDialogVisible: false,
      assignForm: {
        id: '',
        username: '',
        rid: ''
      },
      roleList: []
    }
  },
  methods: {
    async getUserList() {
      // axios如果是get/delete请求，参数要么直接拼地址栏，要么放到params中
      // 如果post/put/patch请求，参数放到data中
      // 除了login请求，其他所有的接口都必须携带token， 要求设置给请求头：Authorization
      let res = await this.axios({
        method: 'get',
        url: 'users',
        params: {
          query: this.query,
          pagenum: this.currentPage,
          pagesize: this.pageSize
        }
      })
      let {
        meta: { status },
        data: { users, total }
      } = res
      if (status === 200) {
        this.userList = users
        this.total = total
      }
    },
    handleSizeChange(val) {
      // 修改this.pageSize
      this.pageSize = val
      this.getUserList()
    },
    handleCurrentChange(val) {
      console.log(val)
      // 把currentPage修改成val
      this.currentPage = val
      this.getUserList()
    },
    search() {
      // 搜索的时候，把当前页第一页
      this.currentPage = 1
      this.getUserList()
    },
    async delUser(id) {
      try {
        await this.$confirm('您确定要删除吗', '温馨提示', {
          type: 'warning'
        })
        // 发送ajax请求，删除数据
        let res = await this.axios({
          method: 'delete',
          url: `users/${id}`
        })
        if (res.meta.status === 200) {
          // 成功了怎么办？
          // 如果我们发现当前页只有一条数据了，应该current减1，渲染上一页
          if (this.userList.length <= 1 && this.currentPage > 1) {
            this.currentPage--
          }
          this.getUserList()
          this.$message.success('恭喜你，删除成功了')
        }
      } catch (e) {
        this.$message.info('取消删除了')
      }
    },
    async changeState({ id, mg_state: mgState }) {
      let res = await this.axios({
        method: 'put',
        url: `users/${id}/state/${mgState}`
      })
      if (res.meta.status === 200) {
        this.$message.success('修改状态成功了')
      } else {
        this.$message.error('修改状态失败了')
      }
    },
    // 显示添加对话框
    showAddDialog() {
      this.addDialogVisible = true
    },
    addUser() {
      // 1. 表单校验功能
      // 2. 发送ajax请求添加数据
      // 3. 重新渲染  关闭模态框  重置样式
      this.$refs.addForm.validate(async valid => {
        if (!valid) return false
        // 校验成功
        let res = await this.axios({
          method: 'post',
          url: 'users',
          data: this.addForm
        })
        let {
          meta: { status, msg }
        } = res
        if (status === 201) {
          this.total++
          this.currentPage = Math.ceil(this.total / this.pageSize)
          // 重新渲染
          this.getUserList()
          // 重置表单样式
          this.$refs.addForm.resetFields()
          // 隐藏模态框
          this.addDialogVisible = false
          // 提示信息
          this.$message.success('添加成功了')
        } else {
          this.$message.error(msg)
        }
      })
    },
    showEditDialog(row) {
      // 显示对话框
      this.editDialogVisible = true
      this.editForm.id = row.id
      this.editForm.username = row.username
      this.editForm.email = row.email
      this.editForm.mobile = row.mobile
    },
    updateUser() {
      this.$refs.editForm.validate(async valid => {
        if (!valid) return false
        // 发送ajax请求
        let res = await this.axios({
          method: 'put',
          url: `users/${this.editForm.id}`,
          data: this.editForm
        })
        let {
          meta: { status }
        } = res
        if (status === 200) {
          // 重新渲染
          this.getUserList()
          // 隐藏修改对话框
          this.editDialogVisible = false
          // 重置修改表单
          this.$refs.editForm.resetFields()
          // 提示消息
          this.$message.success('恭喜你，修改成功了')
        } else {
          this.$message.error('服务器异常')
        }
      })
    },
    async getUserInfo(id) {
      let res = await this.axios.get(`users/${id}`)
      if (res.meta.status === 200) {
        let rid = res.data.rid
        if (rid === -1) {
          rid = ''
        }
        this.assignForm.rid = rid
      }
    },
    async showAssignDialog(user) {
      console.log(user)
      // 1.显示分配角色的对话框
      this.assignDialogVisible = true
      // 2. 数据回显
      this.assignForm.id = user.id
      this.assignForm.username = user.username
      // 发送ajax，根据用户id查询角色id
      this.getUserInfo(user.id)
      // 3. 获取角色列表
      let res = await this.axios.get('roles')
      if (res.meta.status === 200) {
        this.roleList = res.data
        console.log(this.roleList)
      }
    },
    async assignRole() {
      // 1. 表单校验？？？？
      if (!this.assignForm.rid) {
        this.$message.error('请选择一个角色')
        return
      }
      let res = await this.axios.put(`users/${this.assignForm.id}/role`, {
        rid: this.assignForm.rid
      })
      if (res.meta.status === 200) {
        this.$refs.assignForm.resetFields()
        this.assignDialogVisible = false
        this.getUserList()
        this.$message.success('分配角色成功了')
      }
    }
  },
  created() {
    // $router: 整个路由对象  跳转路由
    // $route:  当前的路由   当前路由中的路由  参数
    this.getUserList()
  }
}
</script>

<style lang="less" scoped>
.input-with-select {
  width: 300px;
  margin-bottom: 5px;
}
</style>
