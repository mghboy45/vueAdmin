<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type='primary'>添加角色</el-button>
        </el-col>
      </el-row>
      <el-table
        :data="rolesList"
        stripe
        border
        style="width: 100%">
        <el-table-column
          type="expand">
          <template slot-scope="scope">
            <el-row :class="['bottomBd', index === 0 ? 'topBd' : '', 'middle']" v-for='(item1, index) in scope.row.children' :key='item1.id'>
              <el-col :span='5'>
                <el-tag closable @close='closeTag(scope.row, item1.id)'>{{item1.authName}}</el-tag>
                <i class='el-icon-caret-right'></i>
              </el-col>
              <el-col :span='19'>
                <el-row v-for='(item2, index2) in item1.children' :key='item2.id' :class="[index2 === 0 ? '' : 'topBd', 'middle']">
                  <el-col :span='8'>
                    <el-tag type='success' closable @close='closeTag(scope.row, item2.id)'>{{item2.authName}}</el-tag>
                    <i class='el-icon-caret-right'></i>
                  </el-col>
                    <el-col :span='16'>
                      <el-tag closable type='warning' v-for='item3 in item2.children' :key='item3.id' @close='closeTag(scope.row, item3.id)'>
                        {{item3.authName}}
                      </el-tag>
                    </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column
          type="index"
          label='#'>
        </el-table-column>
        <el-table-column
          prop="roleName"
          label="角色名称"
          width="180">
        </el-table-column>
        <el-table-column
          prop="roleDesc"
          label="角色描述"
          width="180">
        </el-table-column>
        <el-table-column
          label="操作">
          <template slot-scope="scope">
            <el-button size='mini' type="primary" icon="el-icon-edit">编辑</el-button>
            <el-button size='mini' type="danger" icon="el-icon-delete">删除</el-button>
            <el-button size='mini' type="warning" icon="el-icon-setting" @click='settingRights(scope.row)'>分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 分配权限弹窗 -->
    <el-dialog
      title="提示"
      :visible.sync="IsdialogVisible"
      width="60%"
      @close='closeDefKey'>
      <el-tree
        :data="rightsList"
        :props="defaultProps"
        show-checkbox
        node-key='id'
        default-expand-all
        ref='treeRef'
        :default-checked-keys='defKeyAry'></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="IsdialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="AllSettingRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'roles',
  data () {
    return {
      rolesList: [],
      rightsList: [],
      IsdialogVisible: false,
      roleId: '',
      defaultProps: {
        label: 'authName',
        children: 'children'
      },
      defKeyAry: []// 默认勾选的权限
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        this.$message.error('获取角色列表失败')
      }
      console.log('roles:', res)
      this.rolesList = res.data
    },
    async closeTag (row, id) {
      const result = await this.$confirm('确认删除, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      console.log(result)
      if (result === 'cancel') {
        this.$message.info('取消成功')
      }
      const { data: res } = await this.$http.delete(`roles/${row.id}/rights/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('权限删除失败')
      }
      row.children = res.data
      this.$message.success('删除成功')
    },
    settingRights (row) {
      this.roleId = row.id
      this.getRightsList()
      this.getDefKey(row, this.defKeyAry)
      this.IsdialogVisible = true
    },
    async getRightsList () {
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$$message.error('权限列表获取失败')
      }
      this.rightsList = res.data
    },
    // 获取已有的权限
    getDefKey (node, arr) {
      // 判断节点是否有  children 属性，如果有就递归，如果没有就将节点的 id 插入 数组中保存
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getDefKey(item, arr))
    },
    closeDefKey () {
      // 当关闭弹窗时，我们需要清除掉上一次的已有权限，否则，DefKeyAry 会越来越大
      this.defKeyAry = []
    },
    async AllSettingRights () {
      const rids = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedNodes()
      ].join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, {
        rids
      })
      if (res.meta.status !== 200) {
        return this.$message.error('权限设置失败')
      }
      this.getRolesList()
      this.$message.success('权限设置成功')
      this.IsdialogVisible = false
    }
  }
}
</script>

<style scoped>
.el-tag {
  margin: 8px;
}
.topBd {
  border-top: 1px solid #ccc;
}
.bottomBd {
  border-bottom: 1px solid #ccc;
}
.middle {
  display: flex;
  align-items: center;
}
</style>
