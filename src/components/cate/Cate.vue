<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type='primary' @click="addCateBtn">添加分类</el-button>
        </el-col>
      </el-row>
      <tree-table
        :data='cateList'
        :columns='columns'
        :show-index='true'
        index-text='#'
        :border='true'
        :selection-type='false'
        :expand-type='false'
        class='tree'>
        <!-- 是否支付 -->
        <template slot='isOk' slot-scope="scope">
          <i v-if='scope.row.cat_deleted === true' class="el-icon-success" style="color: lightgreen"></i>
          <i v-else class="el-icon-error" style="color: red"></i>
        </template>
        <!-- 排序 -->
        <template slot='level' slot-scope="scope">
          <el-tag v-if='scope.row.cat_level === 0'>一级</el-tag>
          <el-tag v-else-if='scope.row.cat_level === 1' type="success">二级</el-tag>
          <el-tag v-else type="warning">三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot='opt' slot-scope="">
          <el-button type="primary" icon="el-icon-edit" size='mini'>编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size='mini'>删除</el-button>
        </template>
      </tree-table>
      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>
    <!-- 添加商品分类 -->
    <el-dialog
        title="添加分类"
        :visible.sync="addCateDialogVisible"
        @close='cateClose'
        width="50%">
        <el-form ref='addCateFromRef' :rules='addCateFromRules' :model="cateFrom" label-width="80px">
        <el-form-item label='分类名称' prop="cat_name">
          <el-input v-model="cateFrom.cat_name"></el-input>
        </el-form-item>
        <el-form-item label='父级分类' prop="catePid">
          <el-cascader
            v-model="casSelectVal"
            :options="casList"
            :props="casProps"
            size='mini'
            clearable
            @change="handleChangeCas"></el-cascader>
        </el-form-item>
      </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addCateDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addCateOk">确 定</el-button>
        </span>
      </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'cate',
  data () {
    return {
      total: 0,
      addCateDialogVisible: false,
      cateList: [],
      casList: [], // 联级菜单数组
      casSelectVal: [], // 用于存放被选中的分类ID
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      cateFrom: {
        cat_pid: 0, // 父级分类的id
        cat_name: '', // 分类名称
        cat_level: 0 // 分类等级 默认是等级一 0 代表一级 1代表二级 3 代表三级
      },
      casProps: {
        value: 'cat_id',
        label: 'cat_name',
        expandTrigger: 'hover',
        children: 'children'
      },
      columns: [{
        label: '分类名称',
        prop: 'cat_name'
      }, {
        label: '是否支付',
        prop: 'cat_deleted',
        type: 'template',
        template: 'isOk'
      }, {
        label: '排序',
        prop: 'cat_level',
        type: 'template',
        template: 'level'
      }, {
        label: '操作',
        type: 'template',
        template: 'opt'
      }],
      addCateFromRules: {
        cat_name: [
          {
            required: true, message: '请输入分类名称', trigger: 'blur'
          }
        ]
      }
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      console.log('cate:', res)
      this.cateList = res.data.result
      this.total = res.data.total
    },
    handleSizeChange (val) {
      console.log(`每页 ${val} 条`)
      this.queryInfo.pagesize = val
      this.getCateList()
    },
    handleCurrentChange (val) {
      console.log(`当前页: ${val}`)
      this.queryInfo.pagenum = val
      this.getCateList()
    },
    addCateBtn () {
      this.getCasList()
      this.addCateDialogVisible = true
    },
    async getCasList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error('数据获取失败')
      }
      this.casList = res.data
      console.log(res.data)
    },
    handleChangeCas (val) {
      if (this.casSelectVal.length > 0) {
        this.cateFrom.cat_pid = this.casSelectVal[this.casSelectVal.length - 1]
        this.cateFrom.cat_level = this.casSelectVal.length
      } else {
        this.cateFrom.cat_level = 0
        this.cateFrom.cat_pid = 0
      }
    },
    addCateOk () {
      this.$refs.addCateFromRef.validate(async val => {
        if (!val) {
          return
        }
        const { data: res } = await this.$http.post('categories', this.cateFrom)
        if (res.meta.status !== 201) {
          this.$message.error('添加失败')
        }
        this.$message.success('添加成功')
        this.getCateList()
      })
      this.addCateDialogVisible = false
    },
    cateClose () {
      this.$refs.addCateFromRef.resetFields()
      this.casSelectVal = []
      this.cateFrom.cat_pid = 0
      this.cateFrom.cat_level = 0
    }
  }
}
</script>

<style scoped>
.tree {
  margin: 15px 0;
}
</style>>
