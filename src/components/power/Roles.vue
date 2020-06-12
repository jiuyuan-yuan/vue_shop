<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary"
                     @click="addDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data="roleList"
                border
                stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row v-for="(item1,i1) in scope.row.children"
                    :key="item1.id"
                    :class="['bdbottom','vcenter',i1 === 0 ? 'bdtop' :'']">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable
                        @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级、三级权限 -->
              <el-col :span="19">
                <!-- 通过for循环嵌套渲染二级权限 -->
                <el-row v-for="(item2,i2) in item1.children"
                        :key="item2.id"
                        :class="['vcenter',i2 === 0 ? '' :'bdtop']">
                  <el-col :span="6">
                    <el-tag closable
                            @close="removeRightById(scope.row,item2.id)"
                            type="success">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag closable
                            @close="removeRightById(scope.row,item3.id)"
                            v-for="(item3) in item2.children"
                            :key="item3.id"
                            type="warning">
                      {{item3.authName}}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称"
                         prop="roleName"></el-table-column>
        <el-table-column label="角色描述"
                         prop="roleDesc"></el-table-column>
        <el-table-column label="操作"
                         width="300px">
          <template slot-scope="scope">
            <el-button size="mini"
                       type="primary"
                       icon="el-icon-edit"
                       @click="showEditDialog(scope.row.id)">编辑
            </el-button>
            <el-button size="mini"
                       type="danger"
                       icon="el-icon-delete"
                       @click="deleteUserById(scope.row.id)">删除
            </el-button>
            <el-button size="mini"
                       type="warning"
                       @click="showSetRightDialog(scope.row)"
                       icon="el-icon-setting">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加用户对话框 -->
    <el-dialog title="添加角色"
               :visible.sync="addDialogVisible"
               width="30%"
               @close="addDialogClosed">
      <!-- 内容主体区域 -->
      <el-form :model="addForm"
               :rules="addFormRules"
               ref="addFormRef"
               label-width="80px">
        <el-form-item label="角色名"
                      prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述"
                      prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改用户对话框 -->
    <el-dialog title="修改用户信息"
               :visible.sync="editDialogVisible"
               width="30%"
               @close="editDialogClosed">
      <el-form :model="editForm"
               :rules="addFormRules"
               ref="editFormRef"
               label-width="70px">
        <el-form-item label="角色名"
                      prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述"
                      prop="roleDesc">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限对话框 -->
    <el-dialog title="分配"
               @close="setRightDialogClosed"
               :visible.sync="setRightDialogVisible"
               width="50%">
      <!-- 树形控件 -->
      <el-tree :data="rightsList"
               ref="treeRef"
               show-checkbox
               default-expand-all
               node-key="id"
               :default-checked-keys="defKeys"
               :props="treeProps"></el-tree>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 所有角色列表数据
      roleList: [],
      // 控制添加对话框显示与隐藏
      addDialogVisible: false,
      // 控制修改对话框显示与隐藏
      editDialogVisible: false,
      // 添加角色
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      // 所有权限列表
      rightsList: [],
      // tree结构绑定数据
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中节点
      defKeys: [],
      // 当前即将分配权限的id
      roleId: '',
      // 控制分配权限对话框显示
      setRightDialogVisible: false,
      // 查询到的用户信息保存
      editForm: {},
      // 修改表单验证规则
      editFormRuls: {
        roleName: [
          { required: true, message: '请输入用户名称', trigger: 'blur' },
          { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输角色身份', trigger: 'blur' },
          { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
        ]
      },
      // 添加表单验证规则
      addFormRules: {
        roleName: [
          { required: true, message: '请输入用户名称', trigger: 'blur' },
          { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
        ],
        roleDesc: [
          { message: '请输角色身份', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getRoleList()
  },
  methods: {
    // 添加用户
    addUser () {
      this.$refs.addFormRef.validate(async valid => {
        // console.log(valid)
        if (!valid) return
        // 发起网络请求
        const { data: res } = await this.$http.post('roles', this.addForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加用户失败')
        }
        this.$message.success('添加用户成功')
        // 隐藏对话框
        this.addDialogVisible = false
        this.getRoleList()
      })
    },
    // 获取角色列表
    async getRoleList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message('获取角色列表失败')
      }
      this.roleList = res.data
    },
    // 根据id删除对应信息
    async deleteUserById (id) {
      // console.log(id)
      // 弹框询问是否删除
      const confirmResult = await this.$confirm(
        '此操作将永久删除该用户, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
      // 确认删除返回 confirm
      // 取消删除返回 cancel
      if (confirmResult !== 'confirm') return this.$message.info('取消删除')
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除角色失败！')
      }
      this.$message.success('删除角色成功！')
      this.getRoleList()
    },
    // 修改角色信息并提交
    editUserInfo () {
      this.$refs.editFormRef.validate(async valid => {
        // console.log(valid)
        if (!valid) return
        // 发起修改请求
        const { data: res } = await this.$http.put('roles/' + this.editForm.roleId, {
          roleName: this.editForm.roleName,
          roleDesc: this.editForm.roleDesc
        })
        if (res.meta.status !== 200) {
          return this.$message.error('修改失败')
        }
        // 提示修改成功
        this.$message.success('修改成功')
        // 关闭对话框
        this.editDialogVisible = false
        // 刷新数据
        this.getRoleList()
      })
    },
    // 监听添加用户对话框的关闭事件
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    // 编辑用户对话框的关闭事件
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    // 展示编辑用户对话框
    async showEditDialog (id) {
      // console.log(id)
      this.editDialogVisible = true
      // 发起网络请求id
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        this.$message.error('查询用户信息失败')
      }
      this.editForm = res.data
    },
    // 通过id删除对应权限
    async removeRightById (role, rightId) {
      const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除')
      }
      // 发起请求
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }
      // this.getRoleList()
      // 重新获取数据会刷新列表
      // 重新赋值即可
      role.children = res.data
    },
    // 展示权限分配对话框
    async showSetRightDialog (role) {
      // 获取所有权限数据
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('权限列表获取失败')
      }
      // 获取权限数据保存
      this.rightsList = res.data
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 通过递归，获取角色下的三级权限id并保存
    getLeafKeys (node, arr) {
      // 如果当前节点没有children则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item =>
        this.getLeafKeys(item, arr))
    },
    // 监听配权限对话框关闭
    setRightDialogClosed () {
      this.defKeys = []
    },
    // 用户权限分配
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`,
        {
          rids: idStr
        })
      if (res.meta.status !== 200) {
        return this.$message.error('权限分配失败')
      }
      this.getRoleList()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>
