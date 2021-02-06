<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card class="box-card">
      <el-row :gutter="20">
        <el-col :span="10">
          <el-input placeholder="请输入内容" class="input-with-select" v-model='queryInfo.query' clearable @clear='getUserList'>
            <el-button slot="append" icon="el-icon-search" @click='getUserList'></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click='addUsers'>添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 用户列表 -->
      <el-table
        :data="userList"
        border
        style="width: 100%"
        stripe>
        <el-table-column type='index' label="#"></el-table-column>
        <el-table-column prop="username" label="姓名"></el-table-column>
        <el-table-column prop="mobile" label="电话"></el-table-column>
        <el-table-column prop="email" label="邮箱"></el-table-column>
        <el-table-column prop="mg_state" label="状态">
          <template slot-scope="scope">
            <el-switch
            v-model="scope.row.mg_state"
            @change='changeStatus(scope.row)'
            >
          </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-tooltip class="item" effect="dark" content="编辑" placement="top" :enterable='false'>
              <el-button type="primary" icon="el-icon-edit" size='mini' @click='editDialogShow(scope.row)'>编辑</el-button>
            </el-tooltip>
            <el-tooltip class="item" effect="dark" content="删除" placement="top" :enterable='false'>
              <el-button type="danger" icon="el-icon-delete" size='mini' @click='delUser(scope.row.id)'>删除</el-button>
            </el-tooltip>
            <el-tooltip class="item" effect="dark" content="权限" placement="top" :enterable='false'>
              <el-button type="warning" icon="el-icon-setting" size='mini' @click='roleMatch(scope.row)'>分配角色</el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 4, 5]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
      <!-- 添加用户对话框 -->
      <el-dialog
        title="添加用户"
        :visible.sync="addDialogVisible"
        width="50%"
        @close='addDialogClose'>
        <el-form ref='addUserFromRef' :rules='userFromRules' :model="userFrom" label-width="70px">
        <el-form-item prop='username' label='用户名'>
          <el-input v-model="userFrom.username"></el-input>
        </el-form-item>
        <el-form-item prop='password' label='密码'>
          <el-input type='password' v-model="userFrom.password"></el-input>
        </el-form-item>
        <el-form-item prop='email' label='邮箱'>
          <el-input v-model="userFrom.email"></el-input>
        </el-form-item>
        <el-form-item prop='mobile' label='手机号'>
          <el-input v-model="userFrom.mobile"></el-input>
        </el-form-item>
      </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addUserOk">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 编辑弹窗 -->
      <el-dialog
        title="添加用户"
        :visible.sync="editDialogVisible"
        width="50%">
        <el-form ref='editUserFromRef' :rules='editUserFromRules' :model="editFrom" label-width="70px">
        <el-form-item label='用户名' prop="username">
          <el-input v-model="editFrom.username" disabled></el-input>
        </el-form-item>
        <el-form-item label='邮箱' prop="email">
          <el-input v-model="editFrom.email"></el-input>
        </el-form-item>
        <el-form-item label='手机号' prop="mobile">
          <el-input v-model="editFrom.mobile"></el-input>
        </el-form-item>
      </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="cancleEdit">取 消</el-button>
          <el-button type="primary" @click="editUserOk">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
    <!-- 设置用户权限对话框 -->
    <el-dialog
      title="提示"
      :visible.sync="dialogVisible"
      width="50%"
      @close='closeRoleDlog'>
      <div>
        <p>这是用户名称： {{userInfo.username}}</p>
        <p>这是用户角色： {{userInfo.role_name}}</p>
        <p>分配新角色：
          <el-select v-model="selecteVal" placeholder="请选择">
            <el-option
              v-for="item in roleList"
              :key="item.id"
              :label="item.roleName"
              :value="item.id">
            </el-option>
          </el-select>
        </p>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
// 邮箱验证
const checkEmail = (rule, val, cb) => {
  const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
  if (regEmail.test(val)) {
    return cb()
  }
  cb(new Error('请输入合法的邮箱'))
}
// 验证手机号
const checkMobile = (rule, val, cb) => {
  const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
  if (regMobile.test(val)) {
    return cb()
  }
  cb(new Error('请输入正确的手机号'))
}
export default {
  name: 'users',
  data () {
    return {
      userList: [],
      total: 0,
      addDialogVisible: false,
      editDialogVisible: false,
      dialogVisible: false,
      userInfo: {},
      roleList: [],
      selecteVal: '',
      queryInfo: {
        query: '', // 查询参数
        pagenum: 1, // 当前页码
        pagesize: 2 // 每条显示条数
      },
      userFrom: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      editFrom: {},
      userFromRules: {
        username: [
          {
            required: true, message: '请输入用户名', trigger: 'blur'
          }
        ],
        password: [
          {
            required: true, message: '请输入密码', trigger: 'blur'
          }
        ],
        email: [
          {
            required: true, message: '请输入邮箱', trigger: 'blur'
          },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          {
            required: true, message: '请输入手机号', trigger: 'blur'
          },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      editUserFromRules: {
        email: [
          {
            required: true, message: '请输入邮箱', trigger: 'blur'
          },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          {
            required: true, message: '请输入手机号', trigger: 'blur'
          },
          { validator: checkMobile, trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getUserList()
  },
  methods: {
    async getUserList () {
      const { data: res } = await this.$http.get('users', { params: this.queryInfo })
      console.log('res:', res)
      if (res.meta.status !== 200) {
        this.$message.error('获取用户列表失败')
      }
      this.userList = res.data.users
      this.total = res.data.total
    },
    handleSizeChange (val) {
      console.log(`每页 ${val} 条`)
      this.queryInfo.pagesize = val
      this.getUserList()
    },
    handleCurrentChange (val) {
      console.log(`当前页: ${val}`)
      this.queryInfo.pagenum = val
      this.getUserList()
    },
    async changeStatus (row) {
      const { data: res } = await this.$http.put(`users/${row.id}/state/${row.mg_state}`)
      if (res.meta.status !== 200) {
        row.mg_state = !row.mg_state
        this.$message.error('更新用户状态失败')
      }
      this.$message.success('用户状态更新完成')
    },
    addUsers () {
      this.addDialogVisible = true
    },
    addDialogClose () {
      this.$refs.addUserFromRef.resetFields()
    },
    addUserOk () {
      this.$refs.addUserFromRef.validate(async val => {
        if (!val) {
          this.$message.error('添加用户失败')
        }
        const { data: res } = await this.$http.post('users', this.userFrom)
        console.log(res)
        if (res.meta.status !== 201) {
          this.$message.error('获取用户失败')
        }
        this.getUserList()
        this.$message.success('添加用户成功')
        this.addDialogVisible = false
      })
    },
    async editDialogShow (row) {
      this.editDialogVisible = true
      const { data: res } = await this.$http.get(`users/${row.id}`)
      console.log(res)
      if (res.meta.status !== 200) {
        this.$message.error('编辑用户失败')
      }
      this.editFrom = res.data
    },
    cancleEdit () {
      this.$refs.editUserFromRef.resetFields()
      this.editDialogVisible = false
    },
    editUserOk () {
      this.$refs.editUserFromRef.validate(async val => {
        if (!val) {
          return
        }
        console.log(this.editFrom.id)
        const { data: res } = await this.$http.put(`users/${this.editFrom.id}`, {
          email: this.editFrom.email,
          mobile: this.editFrom.mobile
        })
        if (res.meta.status !== 200) {
          this.$message.error('更新用户失败')
        }
        this.editDialogVisible = false
        this.getUserList()
        this.$message.success('更新用户成功')
      })
    },
    async delUser (id) {
      const result = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      console.log(result)
      if (result === 'cancel') {
        this.$message.info('取消成功')
      }
      const { data: res } = await this.$http.delete(`users/${id}`)
      if (res.meta.status !== 200) {
        this.$message.error('删除失败')
      }
      this.$message.success('删除成功')
      this.getUserList()
    },
    async roleMatch (row) {
      this.userInfo = row
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.roleList = res.data
      this.dialogVisible = true
    },
    async saveRoleInfo () {
      if (!this.selecteVal) {
        return this.$message.error('请选择新的角色')
      }
      const { data: res } = await this.$http.put(`users/${this.userInfo.id}/role`, {
        rid: this.selecteVal
      })
      if (res.meta.status !== 200) {
        return this.$message.error('角色设置失败')
      }
      this.getUserList()
      this.$message.success('角色设置成功')
      this.dialogVisible = false
    },
    closeRoleDlog () {
      this.selecteVal = ''
      this.userInfo = {}
    }
  }
}
</script>

<style>

</style>
