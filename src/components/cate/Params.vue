<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-alert
        title="注意：只允许为第三级分类设置属性"
        type="warning"
        :closable="false"
        show-icon>
      </el-alert>
      <el-row class='selectGood'>
        <el-col :span='12'>
          <span>请选择商品分类</span>
          <el-cascader
            v-model="casSelectVal"
            :options="cateList"
            :props="casProps"
            size='mini'
            clearable
            @change="handleChangeCas"></el-cascader>
        </el-col>
      </el-row>
      <el-tabs v-model="activeName" @tab-click="handleParamsClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button type='primary' size="mini" :disabled='isDisabled' @click="showDialog">添加参数</el-button>
          <el-table
            :data="manyTabData"
            stripe
            border
            style="width: 100%">
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag closable v-for='(item, index) in scope.row.attr_vals' :key='index' @close='handleCloseTag(index, scope.row)'>{{item}}</el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column
              type="index"
              label='#'>
            </el-table-column>
            <el-table-column
              prop="attr_name"
              label="参数名称">
            </el-table-column>
            <el-table-column label="操作哦">
              <template slot-scope="scope">
                <el-button size='mini' type="primary" icon="el-icon-edit" @click='editParamsEvent(scope.row.attr_id)'>编辑</el-button>
                <el-button size='mini' type="danger" icon="el-icon-delete" @click="delParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 静态属性 -->
        <el-tab-pane label="静态参数" name="only">
          <el-button type='primary' size="mini" :disabled='isDisabled' @click="showDialog">添加属性</el-button>
          <el-table
            :data="onlyTabData"
            stripe
            border
            style="width: 100%">
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag closable v-for='(item, index) in scope.row.attr_vals' :key='index' @close='handleCloseTag(index, scope.row)'>{{item}}</el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column
              type="index"
              label='#'>
            </el-table-column>
            <el-table-column
              prop="attr_name"
              label="属性名称">
            </el-table-column>
            <el-table-column label="操作哦">
              <template slot-scope="scope">
                <el-button size='mini' type="primary" icon="el-icon-edit" @click='editParamsEvent(scope.row.attr_id)'>编辑</el-button>
                <el-button size='mini' type="danger" icon="el-icon-delete" @click="delParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加参数/属性对话框 -->
    <el-dialog
      :title="`添加${titleTxt}`"
      :visible.sync="IsdialogVisible"
      width="50%"
      @close="handleClose">
      <el-form ref='addParamsFromRef' :rules='addParamsFromRules' :model="addParamsFrom" label-width="80px">
        <el-form-item :label='titleTxt' prop="attr_name">
          <el-input v-model="addParamsFrom.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="IsdialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParamsOk">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑对话框 -->
    <el-dialog
      :title="`编辑${titleTxt}`"
      :visible.sync="editdialogVisible"
      width="50%"
      @close="edithandleClose">
      <el-form ref='editParamsFromRef' :model="editParamsFrom" label-width="80px">
        <el-form-item :label='titleTxt' prop="attr_name">
          <el-input v-model="editParamsFrom.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editdialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParamsOk">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'params',
  data () {
    return {
      activeName: 'many',
      IsdialogVisible: false,
      editdialogVisible: false,
      inputVisible: false,
      inputValue: '',
      cateList: [],
      casSelectVal: [],
      manyTabData: [],
      onlyTabData: [],
      casProps: {
        value: 'cat_id',
        label: 'cat_name',
        expandTrigger: 'hover',
        children: 'children'
      },
      addParamsFrom: {},
      editParamsFrom: {
        attr_name: ''
      },
      addParamsFromRules: {
        attr_name: [
          {
            required: true, message: '请输入名称', trigger: 'blur'
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
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        this.$message.error('分类列表获取失败')
      }
      this.cateList = res.data
    },
    async getParams () {
      const { data: res } = await this.$http.get(`categories/${this.getCurId}/attributes`, { params: { sel: this.activeName } })
      if (res.meta.status !== 200) {
        this.$message.error('获取参数失败')
      }
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        item.inputVisible = false
        item.inputValue = ''
      })
      console.log('123', res.data)
      if (this.activeName === 'many') {
        this.manyTabData = res.data
      } else {
        this.onlyTabData = res.data
      }
      console.log('parmas:', res.data)
    },
    // 当级联发生改变时，获取参数列表
    handleChangeCas () {
      if (this.casSelectVal.length !== 3) {
        this.casSelectVal = []
      }
      this.getParams()
    },
    // 切换Tab 按钮发送请求分别获取动态参数和静态参数
    handleParamsClick () {
      this.getParams()
    },
    showDialog () {
      this.IsdialogVisible = true
    },
    // 关闭对话框清除 from表单 校验规则
    handleClose () {
      this.$refs.addParamsFromRef.resetFields()
      this.addParamsFrom = {}
    },
    // 编辑对话框关闭时清除 from表单校验规则
    edithandleClose () {
      this.$refs.editParamsFromRef.resetFields()
    },
    // 添加参数/属性事件
    addParamsOk () {
      this.$refs.addParamsFromRef.validate(async val => {
        if (!val) return
        const { data: res } = await this.$http.post(`categories/${this.getCurId}/attributes`, {
          attr_name: this.addParamsFrom.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 201) {
          this.$message.error('添加失败')
        }
        this.$message.success('添加成功')
        this.getParams()
      })
      this.IsdialogVisible = false
    },
    // 编辑按钮通过 id  查询
    async editParamsEvent (id) {
      const { data: res } = await this.$http.get(`categories/${this.getCurId}/attributes/${id}`, { params: { attr_sel: this.activeName } })
      if (res.meta.status !== 200) {
        this.$message.error('获取参数失败')
      }
      this.editParamsFrom = res.data
      this.editdialogVisible = true
    },
    // 修改动态参数/静态属性
    editParamsOk () {
      this.$refs.editParamsFromRef.validate(async val => {
        if (!val) {
          return null
        }
        const { data: res } = await this.$http.put(`categories/${this.getCurId}/attributes/${this.editParamsFrom.attr_id}`, {
          attr_name: this.editParamsFrom.attr_name,
          attr_sel: this.activeName
        })
        console.log(res.data)
        if (res.meta.status !== 200) {
          this.$message.error('修改失败')
        }
        this.getParams()
        this.$message.success('修改成功')
      })
      this.editdialogVisible = false
    },
    // 根据指定 ID 删除 属性
    async delParams (id) {
      const result = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (result === 'cancel') {
        return this.$message.info('取消成功')
      }
      const { data: res } = await this.$http.delete(`categories/${this.getCurId}/attributes/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }
      console.log('delparams', res.data)
      this.getParams()
      this.$message.success('删除成功')
    },
    handleInputConfirm (row) {
      const inputValue = row.inputValue
      if (inputValue.trim().length === 0) {
        return this.$message.error('请输入合法的内容')
      }
      row.attr_vals.push(inputValue)
      row.inputVisible = false
      row.inputValue = ''
      this.setTagInput(row)
    },
    async setTagInput (row) {
      const { data: res } = await this.$http.put(`categories/${this.getCurId}/attributes/${row.attr_id}`, {
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(' ')
      })
      if (res.meta.status !== 200) {
        return this.$message.error('修改失败')
      }
      this.$message.success('修改成功')
    },
    showInput (row) {
      row.inputVisible = true
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    handleCloseTag (index, row) {
      row.attr_vals.splice(index, 1)
      this.setTagInput(row)
    }
  },
  computed: {
    isDisabled () {
      if (this.casSelectVal.length !== 3) {
        return true
      }
      return false
    },
    getCurId () {
      if (this.casSelectVal.length === 3) {
        return this.casSelectVal[this.casSelectVal.length - 1]
      }
      return null
    },
    titleTxt () {
      if (this.activeName === 'many') {
        return '参数名称'
      }
      return '属性名称'
    }
  }
}
</script>

<style scoped>
.selectGood {
  margin-bottom: 15px;
}
.el-tag {
  margin: 5px;
}
.input-new-tag {
  width: 90px;
}
</style>>
