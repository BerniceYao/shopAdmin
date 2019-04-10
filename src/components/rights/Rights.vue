<template>
  <div class="rights">
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 表格组件显示权限数据 -->
    <el-table :data="rightList" style="width: 100%;">
      <!-- 每一列 -->
      <el-table-column type="index"></el-table-column>
      <el-table-column label="权限名称" prop="authName"></el-table-column>
      <el-table-column label="路径" prop="path"></el-table-column>
      <el-table-column label="层级" prop="level">
        <template slot-scope="{row: {level}}">
          <span v-if="+level === 0">一级</span>
          <span v-else-if="+level === 1">二级</span>
          <span v-else>三级</span>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
/*
  权限列表功能展示
    1. 获取所有的权限列表数据
    2. 通过table把所有的权限列表展示出来
*/
export default {
  data() {
    return {
      rightList: []
    }
  },
  methods: {
    // 获取所有权限列表的数据
    async getRightList() {
      let res = await this.axios.get('rights/list')
      let {
        meta: { status },
        data
      } = res
      if (status === 200) {
        this.rightList = data
        // console.log(this.rightList)
      }
    }
  },
  created() {
    this.getRightList()
  }
}
</script>

<style>
</style>
