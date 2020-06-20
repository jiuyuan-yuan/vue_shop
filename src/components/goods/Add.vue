<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 提示区域 -->
      <el-alert title="添加商品信息"
                type="info"
                show-icon
                :closable="false">
      </el-alert>
      <!-- 步骤条区域 -->
      <el-steps :space="200"
                :active="activeStep-0"
                finish-status="success"
                align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>

      <!-- 表单区域 -->
      <el-form :model="addForm"
               :rules="addFormRules"
               ref="ruleFormRef"
               label-position="top"
               label-width="100px">

        <!-- tab切换栏 -->
        <el-tabs v-model="activeStep"
                 :before-leave="beforeTabLeave"
                 @tab-click="tabClicked"
                 :tab-position="'left'">
          <el-tab-pane label="基本信息"
                       name="0">
            <el-form-item label="商品名称"
                          prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格"
                          prop="goods_price">
              <el-input v-model="addForm.goods_price"
                        type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量"
                          prop="goods_weight">
              <el-input v-model="addForm.goods_weight"
                        type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量"
                          prop="goods_number">
              <el-input v-model="addForm.goods_number"
                        type="number"></el-input>
            </el-form-item>

            <!-- 级联选择器 -->
            <el-form-item label="商品分类"
                          prop="goods_cat">
              <el-cascader v-model="addForm.goods_cat"
                           :options="catelist"
                           :props="cateProps"
                           @change="handleChange"></el-cascader>

            </el-form-item>

          </el-tab-pane>
          <el-tab-pane label="商品参数"
                       name="1">
            <!-- 表单渲染item项 -->
            <el-form-item :label="item.attr_name"
                          v-for="item in manyTableData"
                          :key="item.attr_id">
              <!-- 复选框组 -->
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox :label="cb"
                             v-for="(cb,i) in item.attr_vals"
                             :key="i"
                             border></el-checkbox>
              </el-checkbox-group>
            </el-form-item>

          </el-tab-pane>
          <el-tab-pane label="商品属性"
                       name="2">
            <el-form-item :label="item.attr_name"
                          v-for="item in onlyTableData"
                          :key="item.attr_id">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片"
                       name="3">
            <!-- 图片上传 -->
            <el-upload class="upload-demo"
                       :headers="headerObj"
                       :action="uploadUrl"
                       :on-preview="handlePreview"
                       :on-remove="handleRemove"
                       :on-success="handleSuccsee"
                       list-type="picture">
              <el-button size="small"
                         type="primary">点击上传</el-button>
              <div slot="tip"
                   class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容"
                       name="4">
            <!-- 富文本编辑组件 -->
            <quill-editor v-model="addForm.goods_introduce">
            </quill-editor>
            <!-- 添加按钮 -->
            <el-button type="primary"
                       class="btnAdd"
                       @click="add">添加商品</el-button>
          </el-tab-pane>
        </el-tabs>

      </el-form>

    </el-card>
    <!-- 图片预览对话框 -->
    <el-dialog title="图片预览"
               :visible.sync="previewVisible"
               width="50%">
      <img :src="previewPath"
           class="preview"
           alt="">
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'

export default {
  data () {
    return {
      // tab和进度条切换索引
      activeStep: '0',
      // 添加的表单
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_number: 0,
        goods_weight: 0,
        // 商品所属分类
        goods_cat: [],
        // 图片数组
        pics: [],
        // 商品详情描述
        goods_introduce: '',
        attrs: []
      },
      // 验证规则
      addFormRules: {
        goods_name: [
          { required: true, message: '请输入名称', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入名称', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入名称', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入重量', trigger: 'blur' }
        ],
        goods_cat: [
          { required: true, message: '请输选择商品分类', trigger: 'blur' }
        ]
      },
      // 商品分类列表
      catelist: [],
      // 级联选择器配置
      cateProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 上传图片的地址
      uploadUrl: 'http://127.0.0.1:8888/api/private/v1/upload',
      // 动态参数列表数组
      manyTableData: [],
      // 静态属性面板
      onlyTableData: [],
      // 请求头设置
      headerObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      // 图片预览地址
      previewPath: '',
      // 图片预览控制
      previewVisible: false
    }
  },
  computed: {
    cateId () {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    // 获取所有商品分类数据
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表失败')
      }
      // console.log(res.data)
      this.catelist = res.data
    },
    // 级联选择器变化触发
    handleChange () {
      // console.log(this.addForm.goods_cat)
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
      // console.log(this.cateId)
    },
    beforeTabLeave (activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
        this.$message.error('请选择商品分类')
        return false
      }
    },
    // tabsd点击事件
    async tabClicked () {
      // console.log(this.activeStep)
      // 证明访问的是动态参数面板
      if (this.activeStep === '1') {
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
          params: { sel: 'many' }
        })
        if (res.meta.status !== 200) {
          return this.$message.error('动态参数获取失败')
        }
        // console.log(res.data)
        res.data.forEach(item => {
          item.attr_vals =
            item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.manyTableData = res.data
        // console.log(this.manyTableData)
      }
      // 证明访问的是静态属性面板
      if (this.activeStep === '2') {
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
          params: { sel: 'only' }
        })
        if (res.meta.status !== 200) {
          return this.$message.error('静态属性获取失败')
        }
        this.onlyTableData = res.data
        // console.log(this.onlyTableData)
      }
    },
    // 图片预览
    handlePreview (file) {
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    // 删除图片
    handleRemove (file) {
      // 找到图片路劲
      const filePath = file.response.data.tmp_path
      // 取出索引值
      const i = this.addForm.pics.findIndex(x => {
        x.pic = filePath
      })
      this.addForm.pics.splice(i, 1)
      console.log(this.addForm.pics)
    },
    // 监听图片上传事件
    handleSuccsee (res) {
      console.log(res)
      // 1 创建图片对象
      // 2 push到pics中
      const picInfo = { pic: res.data.tmp_path }
      this.addForm.pics.push(picInfo)
    },
    // 添加商品函数
    add () {
      this.$refs.ruleFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('请填写商品必要项')
        }
        // 执行添加的业务逻辑
        const form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join(',')
        // 处理动态参数
        this.manyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_vals: item.attr_vals.join(' ')
          }
          this.addForm.attrs.push(newInfo)
        })
        // 处理静态属性
        this.onlyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_vals: item.attr_vals
          }
          this.addForm.attrs.push(newInfo)
        })
        form.attrs = this.addForm.attrs
        // console.log(form)
        // 添加商品
        const { data: res } = await this.$http.post('goods', form)
        console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error('添加失败')
        }
        this.$message.success('添加成功')
        this.$router.push('/goods')
      })
    }
  }
}
</script>

<style lang="less" scoped>
.el-checkbox-group {
  margin: 0 10px 0 0;
}

.preview {
  width: 100%;
}

.btnAdd {
  margin-top: 15px;
}
</style>
