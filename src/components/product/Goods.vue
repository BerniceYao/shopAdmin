<template>
  <div class="goods">
    <el-button type="success" plain @click="$router.push('/goods-add')">添加商品</el-button>
    <el-table :data="goodList">
      <el-table-column type="index" :index="indexMethod"></el-table-column>
      <el-table-column label="商品名称" prop="goods_name"></el-table-column>
      <el-table-column label="商品价格" prop="goods_price"></el-table-column>
      <el-table-column label="商品重量" prop="goods_weight"></el-table-column>
      <el-table-column label="创建时间" prop="add_time">
        <template slot-scope="{row}">{{row.add_time | dateFilter}}</template>
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="{row}">
          <el-button type="primary" icon="el-icon-edit" size="mini" plain></el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini" plain></el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage"
      :page-sizes="[10, 20, 30, 40]"
      :page-size="pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
      background
    ></el-pagination>
  </div>
</template>

<script>
export default {
  data() {
    return {
      goodList: [],
      currentPage: 1,
      pageSize: 10,
      total: 0,
      query: ''
    }
  },
  methods: {
    async getGoodList() {
      let res = await this.axios.get('goods', {
        params: {
          query: this.query,
          pagenum: this.currentPage,
          pagesize: this.pageSize
        }
      })
      let {
        meta: { status },
        data: { total, goods }
      } = res
      if (status === 200) {
        this.goodList = goods
        this.total = total
        console.log(this.goodList)
      }
    },
    handleSizeChange(val) {
      this.pageSize = val
      this.getGoodList()
    },
    handleCurrentChange(val) {
      this.currentPage = val
      this.getGoodList()
    },
    indexMethod(index) {
      return index + 1 + (this.currentPage - 1) * this.pageSize
    }
  },
  created() {
    this.getGoodList()
  }
}
</script>

<style>
</style>
