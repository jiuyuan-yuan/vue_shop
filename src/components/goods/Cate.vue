<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片识图区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary"
                     @click="showAddCateDialog">
            添加分类
          </el-button>
        </el-col>
      </el-row>
      <!-- 表格 -->
      <tree-table class="tree-table"
                  :data="cateList"
                  :columns="columns"
                  :selection-type="false"
                  show-index
                  index-text="#"
                  :expand-type="false"
                  border
                  :show-row-hover="false">
        <!-- 是否有效 -->
        <template slot="isok"
                  slot-scope="scope">
          <i class="el-icon-success"
             v-if="scope.row.cat_deleted === false"
             style="color:lightgreen"></i>
          <i class="el-icon-error"
             v-else
             style="color:red"></i>
        </template>
        <!-- 排序 -->
        <template slot="order"
                  slot-scope="scope">
          <el-tag size="mini"
                  v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag size="mini"
                  type="success"
                  v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag size="mini"
                  type="warning"
                  v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt"
                  slot-scope="scope">
          <el-button type="primary"
                     size="mini"
                     icon="el-icon-edit"
                     @click="showEidtCateName(scope.row)">编辑</el-button>
          <el-button type="danger"
                     icon="el-icon-delete"
                     size="mini"
                     @click="deleteCate(scope.row.cat_id)">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange"
                     @current-change="handleCurrentChange"
                     :current-page="queryInfo.pagenum"
                     :page-sizes="[3, 5, 10, 15]"
                     :page-size="queryInfo.pagesize"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="total">
      </el-pagination>
    </el-card>
    <!-- 添加分类对话框 -->
    <el-dialog title="添加分类"
               :visible.sync="addDialogVisible"
               width="30%"
               @close="addCateDialogClosed">
      <!-- 添加分类的表单 -->
      <el-form :model="addCateForm"
               :rules="addCateFormRules"
               ref="addCateFormRef"
               label-width="100px">
        <el-form-item label="分类名称:"
                      prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类:">
          <!-- options用来指定数据源 -->
          <el-cascader v-model="selectedKeys"
                       :options="parentCateList"
                       :props="cascaderProps"
                       @change="parentCateChange"
                       clearable></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑名称对话框 -->
    <el-dialog title="编辑"
               :visible.sync="editDialogVisible"
               width="30%"
               @close="editCateDialogClosed">
      <el-form :model="editCateForm"
               :rules="editCateFormRules"
               ref="editCateFormRef"
               label-width="100px">
        <el-form-item label="商品名称"
                      prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="editCateInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分类的数据列表,默认空
      cateList: [],
      // 总数据条数
      total: 0,
      columns: [{
        label: '分类名称',
        prop: 'cat_name'
      }, {
        label: '是否有效',
        // 自定义模版
        type: 'template',
        // 表当前这一列模版使用名称
        template: 'isok'
      }, {
        label: '排序',
        // 自定义模版
        type: 'template',
        // 表当前这一列模版使用名称
        template: 'order'
      }, {
        label: '操作',
        // 自定义模版
        type: 'template',
        // 表当前这一列模版使用名称
        template: 'opt'
      }],
      // 控制添加分类对话框显示隐藏
      addDialogVisible: false,
      // 编辑名称对话框显示隐藏
      editDialogVisible: false,
      // 将要添加分类表单数据对象
      addCateForm: {
        // 将要添加的分类名称
        cat_name: '',
        // 父级分类的id
        cat_pid: 0,
        // 分类等级，默认添加一级分类
        cat_level: 0
      },
      // 父级分类的列表
      parentCateList: [],
      // 制定级联选择权配置对象
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover',
        checkStrictly: 'true'
      },
      // 选中的父级分类id数组
      selectedKeys: [],
      // 查询到的表单
      editCateForm: {},
      // 将要添加的验证规则对象
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' },
          { min: 2, max: 6, message: '长度在 2 到 6 个字符', trigger: 'blur' }
        ]
      },
      editCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' },
          { min: 2, max: 6, message: '长度在 2 到 6 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    // 获取商品分类数据
    this.getCateList()
  },
  methods: {
    // 获取商品数据
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('查询参数失败')
      }
      // console.log(res.data)
      // 数据列表更新
      this.cateList = res.data.result
      // 为总数据条数赋值
      this.total = res.data.total
    },
    // 监听pagesize改变事件
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听pagenum改变
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 获取父级分类数据列表
    showAddCateDialog () {
      this.addDialogVisible = true
      this.getParentCateList()
    },
    // 获取父级分类的数据列表
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error('查询参数失败')
      }
      this.parentCateList = res.data
    },
    // 选择项变化触发
    parentCateChange () {
      // 如果选中值长度 > 0 则选中父级分类
      // 反之 没有选择任何父级分类i
      if (this.selectedKeys.length > 0) {
        // 父级分类id
        this.addCateForm.cat_pid = this.selectedKeys[
          this.selectedKeys.length - 1
        ]
        // 当前分类等级赋值
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        // 父级分类id
        this.addCateForm.cat_pid = 0
        // 当前分类等级赋值
        this.addCateForm.cat_level = 0
      }
    },
    // 点击按钮添加分类
    addCate () {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加失败')
        }
        this.$message.success('添加成功')
        this.getCateList()
        this.addDialogVisible = false
      })
    },
    // 对话框关闭重置表单
    addCateDialogClosed () {
      this.selectedKeys = 0
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
      this.$refs.addCateFormRef.resetFields()
    },
    // 展示编辑展示对话框
    async showEidtCateName (row) {
      // console.log(row.cat_id)
      const { data: res } = await this.$http.get('categories/' + row.cat_id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询参数失败')
      }
      // console.log(res)
      this.editCateForm = res.data
      this.editDialogVisible = true
    },
    // 对话框关闭重置表单
    editCateDialogClosed () {
      this.$refs.editCateFormRef.resetFields()
    },
    // 编辑信息  无法编辑
    editCateInfo () {
      this.$refs.editCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put('categories/' + this.editCateForm.cat_id,
          this.editCateForm.cat_name)
        // console.log('categories/' + this.editCateForm.cat_id)
        // console.log(this.editCateForm.cat_id, this.editCateForm.cat_name)
        if (res.meta.status !== 200) {
          return this.$message.error('修改失败')
        }
        this.$message.success('修改成功')
      })
      this.getCateList()
      this.editDialogVisible = false
    },
    // 根据id删除
    async deleteCate (id) {
      const confirmResult = await this.$confirm(
        '确认删除?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // 确认删除返回 confirm
      // 取消删除返回 cancel
      if (confirmResult !== 'confirm') return this.$message.info('取消删除')
      const { data: res } = await this.$http.delete('categories/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败！')
      }
      this.$message.success('删除成功！')
      this.getCateList()
    }
  }
}
</script>

<style lang="less" scoped>
.tree-table {
  margin-top: 15px;
}
.el-cascader {
  width: 100%;
}
</style>
