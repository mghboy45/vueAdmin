<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-table
        :data="rightsList"
        stripe
        border
        style="width: 100%">
        <el-table-column
          type="index"
          label='#'>
        </el-table-column>
        <el-table-column
          prop="authName"
          label="权限管理"
          width="180">
        </el-table-column>
        <el-table-column
          prop="path"
          label="路径"
          width="180">
        </el-table-column>
        <el-table-column
          prop="level"
          label="权限等级">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.level === '0'">等级一</el-tag>
            <el-tag v-else-if="scope.row.level === '1'" type="success">等级二</el-tag>
            <el-tag v-else type="warning">等级三</el-tag>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  name: 'rights',
  data () {
    return {
      rightsList: [],
      total: 0,
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 1
      }
    }
  },
  created () {
    this.getRightsList()
  },
  methods: {
    async getRightsList () {
      const { data: res } = await this.$http.get('rights/list')
      console.log('rights:', res)
      if (res.meta.status !== 200) {
        this.$message.error('获取权限列表失败')
      }
      this.rightsList = res.data
    }
  }
}
</script>

<style>

</style>
