<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row :gutter="20">
        <el-col :span="10">
          <el-input placeholder="请输入内容" class="input-with-select" v-model='queryInfo.query' clearable @clear='getGoodsList'>
            <el-button slot="append" icon="el-icon-search" @click='getGoodsList'></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click='addGoods'>添加用户</el-button>
        </el-col>
      </el-row>
      <!-- table表格 -->
      <el-table
        :data="goodsList"
        border
        style="width: 100%"
        stripe>
        <el-table-column type='index' label="#"></el-table-column>
        <el-table-column prop="goods_name" label="商品名称"></el-table-column>
        <el-table-column prop="goods_price" label="商品价格"></el-table-column>
        <el-table-column prop="goods_weight" label="商品重量"></el-table-column>
        <el-table-column prop="add_time" label="创建时间">
          <template slot-scope="scope">
            {{scope.row.add_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-tooltip class="item" effect="dark" content="编辑" placement="top" :enterable='false'>
              <el-button type="primary" icon="el-icon-edit" size='mini'>编辑</el-button>
            </el-tooltip>
            <el-tooltip class="item" effect="dark" content="删除" placement="top" :enterable='false'>
              <el-button type="danger" icon="el-icon-delete" size='mini' @click="delById(scope.row.goods_id)">删除</el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  name: 'goods',
  data () {
    return {
      total: 0,
      goodsList: [],
      queryInfo: {
        query: '', // 查询参数
        pagenum: 1, // 当前页码
        pagesize: 10 // 每条显示条数
      }
    }
  },
  created () {
    this.getGoodsList()
  },
  methods: {
    async getGoodsList () {
      const { data: res } = await this.$http.get('goods', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('数据获取失败')
      }
      console.log(res)
      this.goodsList = res.data.goods
      this.total = res.data.total
    },
    addGoods () {
      this.$router.push('/goods/add')
    },
    handleSizeChange (val) {
      console.log(`每页 ${val} 条`)
      this.queryInfo.pagesize = val
      this.getGoodsList()
    },
    handleCurrentChange (val) {
      console.log(`当前页: ${val}`)
      this.queryInfo.pagenum = val
      this.getGoodsList()
    },
    async delById (id) {
      const result = await this.$confirm('此操作将永久删除, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (result === 'cancel') {
        return this.$message.info('取消成功')
      }
      const { data: res } = await this.$http.delete(`goods/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }
      this.$message.success('删除成功')
      this.getGoodsList()
    }
  }
}
</script>

<style>

</style>
