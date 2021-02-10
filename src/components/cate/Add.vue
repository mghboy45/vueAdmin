<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-alert
        title="消息提示的文案"
        type="info"
        show-icon
        :closable='false'
        center>
      </el-alert>
      <!-- 步骤条 -->
      <el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <el-form :model="addGoodsForm" :rules="addGoodsrules" ref="addGoodsFormRef" label-width="100px" label-position='top'>
        <el-tabs v-model="activeIndex" :tab-position="'left'" :before-leave='beforeLeave'
        @tab-click='tabClick'>
          <el-tab-pane label="基本信息" name='0'>
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addGoodsForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addGoodsForm.goods_price"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addGoodsForm.goods_number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addGoodsForm.goods_weight"></el-input>
            </el-form-item>
            <el-form-item label='商品分类' prop="goods_cat">
              <el-cascader
                v-model="addGoodsForm.goods_cat"
                :options="cateList"
                :props="cateProps"
                size='mini'
                clearable
                @change="handleChangeCate"></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name='1'>
            <el-form-item :label="item.attr_name" v-for='item in manyDataList' :key='item.attr_id'>
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox border :label="item2" v-for='(item2, index) in item.attr_vals' :key='index'></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name='2'>
            <el-form-item :label="item.attr_name" v-for='item in onlyCateData' :key='item.attr_id'>
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name='3'>
            <el-form-item>
              <el-upload
                class="upload-demo"
                action="http://127.0.0.1:8888/api/private/v1/upload"
                :on-preview="handlePreview"
                :on-remove="handleRemove"
                list-type="picture"
                :headers='headersObj'
                :on-success='handleUpdataImg'>
                <el-button size="small" type="primary">点击上传</el-button>
              </el-upload>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name='4'>
            <quill-editor v-model="addGoodsForm.goods_introduce"></quill-editor>
            <el-button class='btn' type='primary' @click='addGoodsBtn'>添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
    <!-- 图片预览 -->
    <el-dialog
      title="图片预览"
      :visible.sync="dialogVisible"
      width="50%">
      <img :src="imgSrc" alt="" class='imgSize'>
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'
export default {
  name: 'add',
  data () {
    return {
      activeIndex: '0',
      dialogVisible: false,
      cateList: [],
      manyDataList: [],
      onlyCateData: [],
      imgSrc: '',
      addGoodsForm: {
        goods_name: '',
        goods_price: 0,
        goods_number: 0,
        goods_weight: 0,
        goods_introduce: '',
        goods_cat: [],
        pics: [],
        attrs: []
      },
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        expandTrigger: 'hover',
        children: 'children'
      },
      headersObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      addGoodsrules: {
        goods_name: [
          {
            required: true, message: '请输入商品名称', trigger: 'blur'
          }
        ],
        goods_price: [
          {
            required: true, message: '请输入商品价格', trigger: 'blur'
          }
        ],
        goods_number: [
          {
            required: true, message: '请输入商品数量', trigger: 'blur'
          }
        ],
        goods_weight: [
          {
            required: true, message: '请输入商品重量', trigger: 'blur'
          }
        ],
        goods_cat: [
          {
            required: true, message: '请选择商品分类', trigger: 'blur'
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
        return this.$message.error('获取失败')
      }
      this.cateList = res.data
    },
    handleChangeCate () {
      console.log(this.addGoodsForm)
    },
    beforeLeave (activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addGoodsForm.goods_cat.length !== 3) {
        this.$message.error('请选择商品分类')
        return false
      }
    },
    async tabClick () {
      // console.log(this.activeIndex)
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: 'many' } })
        if (res.meta.status !== 200) {
          return this.$message.error('获取失败')
        }
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        })
        this.manyDataList = res.data
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: 'only' } })
        if (res.meta.status !== 200) {
          return this.$message.error('获取失败')
        }
        this.onlyCateData = res.data
      }
    },
    handlePreview (file) {
      console.log(file)
      this.imgSrc = file.response.data.url
      this.dialogVisible = true
    },
    handleRemove (file) {
      console.log(file)
      const temPath = file.response.data.tmp_path
      const index = this.addGoodsForm.pics.findIndex(item => item.pics === temPath)
      this.addGoodsForm.pics.splice(index, 1)
      console.log(this.addGoodsForm)
    },
    handleUpdataImg (res) {
      const imgPath = { pics: res.data.tmp_path }
      this.addGoodsForm.pics.push(imgPath)
    },
    addGoodsBtn () {
      this.$refs.addGoodsFormRef.validate(async val => {
        if (!val) {
          return this.$message.error('请输入必填信息')
        }
        const newGoodsFrom = _.cloneDeep(this.addGoodsForm)
        newGoodsFrom.goods_cat = newGoodsFrom.goods_cat.join(',')
        // 处理动态参数
        this.manyDataList.forEach(item => {
          const attrsInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' ')
          }
          this.addGoodsForm.attrs.push(attrsInfo)
        })
        // 处理静态属性
        this.onlyCateData.forEach(item => {
          const attrsInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals
          }
          this.addGoodsForm.attrs.push(attrsInfo)
        })
        newGoodsFrom.attrs = this.addGoodsForm.attrs
        const { data: res } = await this.$http.post('goods', newGoodsFrom)
        if (res.meta.status !== 201) {
          return this.$message.error('添加失败')
        }
        this.$message.success('添加成功')
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    cateId () {
      if (this.addGoodsForm.goods_cat.length === 3) {
        return this.addGoodsForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>

<style scoped>
.el-checkbox {
  margin: 0 10px 0 0 !important;
}
.imgSize {
  width: 100%;
}
.btn {
  margin-top: 10px;
}
</style>>
