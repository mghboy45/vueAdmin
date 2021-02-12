<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col :span='12'>
          <el-input placeholder="请输入内容" class="input-with-select" v-model='queryInfo.query' clearable @clear='getOrderList'>
            <el-button slot="append" icon="el-icon-search" @click='getOrderList'></el-button>
          </el-input>
        </el-col>
      </el-row>
      <el-table
        :data="orderList"
        border
        style="width: 100%"
        stripe>
        <el-table-column type='index' label="#"></el-table-column>
        <el-table-column prop="order_number" label="订单编号"></el-table-column>
        <el-table-column prop="order_price" label="订单价格"></el-table-column>
        <el-table-column prop="order_pay" label="是否付款">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.order_pay === '1'" type='success'>已付款</el-tag>
          <el-tag v-else type='danger'>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="is_send" label="是否发货"></el-table-column>
        <el-table-column prop="create_time" label="下单时间">
          <template slot-scope="scope">
            {{scope.row.create_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template>
            <el-button type="primary" icon="el-icon-edit" size='mini' @click="handleAdress">编辑</el-button>
            <el-button type="success" icon="el-icon-location" size='mini' @click="handleLogInfo">物流信息</el-button>
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
    <!-- 地址 -->
    <el-dialog
      title="修改地址"
      :visible.sync="isAddress"
      width="50%"
      @close='handleClose'>
      <el-form ref='editAddressFromRef' :rules='editAddressFromRules' :model="editAddressFrom" label-width="100px">
        <el-form-item label='省市区/县' prop="address1">
          <el-cascader
            v-model="selectCity"
            :options="cityData"
            :props="cateProps"
            size='mini'
            clearable></el-cascader>
        </el-form-item>
        <el-form-item label='详细地址' prop="address2">
          <el-input v-model="editAddressFrom.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="isAddress = false">取 消</el-button>
        <el-button type="primary" @click="isAddress = false">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 物流信息 -->
    <el-dialog
      title="修改地址"
      :visible.sync="isLogInfo"
      width="50%">
      <el-timeline>
        <el-timeline-item
          v-for="(activity, index) in logInfoList"
          :key="index"
          :timestamp="activity.time">
            {{activity.context}}
        </el-timeline-item>
      </el-timeline>
    </el-dialog>
  </div>
</template>

<script>
import cityData from './citydata'
import logInfoData from './logdata'
export default {
  name: 'order',
  data () {
    return {
      total: 0,
      isAddress: false,
      isLogInfo: false,
      orderList: [],
      selectCity: [],
      logInfoList: logInfoData.data,
      cityData,
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      editAddressFrom: {
        address2: '',
        address1: []
      },
      cateProps: {
        value: 'value',
        label: 'label',
        expandTrigger: 'hover',
        children: 'children'
      },
      editAddressFromRules: {
        address1: [
          {
            required: true, message: '请选择省市/县', trigger: 'blur'
          }
        ],
        address2: [
          {
            required: true, message: '请输入详细地址', trigger: 'blur'
          }
        ]
      }
    }
  },
  created () {
    this.getOrderList()
  },
  methods: {
    async getOrderList () {
      const { data: res } = await this.$http.get('orders', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('订单数据获取失败')
      }
      this.total = res.data.total
      this.orderList = res.data.goods
    },
    handleSizeChange (val) {
      console.log(`每页 ${val} 条`)
      this.queryInfo.pagesize = val
      this.getOrderList()
    },
    handleCurrentChange (val) {
      console.log(`当前页: ${val}`)
      this.queryInfo.pagenum = val
      this.getOrderList()
    },
    handleAdress () {
      this.isAddress = true
    },
    handleClose () {
      this.$refs.editAddressFromRef.resetFields()
      this.selectCity = []
    },
    async handleLogInfo () {
      this.isLogInfo = true
      console.log(this.logInfoList)
    }
  }
}
</script>

<style scoped>
.el-cascader {
  width: 100%;
}
</style>
