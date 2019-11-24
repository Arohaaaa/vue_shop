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
            <!-- 分配角色按钮区域 -->
            <el-row>
                <el-col><el-button type="primary" @click="addDialogVisible=true">分配角色</el-button></el-col>
            </el-row>

            <!-- 角色列表区域 -->
            <el-table :data="rolesList">
                <!-- 展开列 -->
                <el-table-column type="expand">
                    <template slot-scope="scope">
                        <el-row :class="['bdbottom', i1==0? 'bdtop': '','vcenter']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
                            <!-- 渲染一级权限 -->
                            <el-col :span="5">
                                <el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <!-- 渲染二级和三级权限 -->
                            <el-col :span="19">
                                <!-- 通过for循环嵌套渲染二级权限 -->
                                <el-row  :class="['bdtop',i2==0?'':'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                                    <el-col :span="6">
                                        <el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">{{item2.authName}}</el-tag>
                                        <i class="el-icon-caret-right"></i>
                                    </el-col>
                                    <el-col :span="18">
                                        <el-tag type="warning" v-for = "(item3) in item2.children" :key="item3.id" closable @close="removeRightById(scope.row,item3.id)">
                                            {{item3.authName}}
                                        </el-tag>
                                    </el-col>
                                </el-row>
                            </el-col>
                        </el-row>
                    </template>
                </el-table-column>
                <el-table-column type="index"></el-table-column>
                <el-table-column label="角色名称" prop="roleName"></el-table-column>
                <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
                <el-table-column label="操作">
                    <template slot-scope="scope">
                        <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditRoleForm(scope.row.id)">编辑</el-button>
                        <el-button size="mini" type="danger" icon="el-icon-delete" @click="delRole(scope.row.id)">删除</el-button>
                        <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                    </template>
                </el-table-column>

            </el-table>
        </el-card>

         <!-- 添加角色对话框 -->
        <el-dialog
            title="添加角色"
            :visible.sync="addDialogVisible"
            width="30%"
            @close="addDialogClosed">
            <!-- 内容主体区域 -->
            <el-form :model="addForm" :rules="addRoleRules"
            label-width="80px" ref="addFormRef">
                <el-form-item label="角色名称" prop="roleName">
                    <el-input v-model="addForm.roleName"></el-input>
                </el-form-item>
                <el-form-item label="角色描述" prop="roleDesc">
                    <el-input v-model="addForm.roleDesc"></el-input>
                </el-form-item>
            </el-form>
            <!-- 底部区域 -->
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addRole">确 定</el-button>
            </span>
        </el-dialog>

          <!-- 编辑角色对话框 -->
        <el-dialog
            title="分配权限"
            :visible.sync="setRightDialogVisible"
            width="30%">
            <!-- 内容主体区域 -->

            <!-- 底部区域 -->
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="allotRights">确 定</el-button>
            </span>
        </el-dialog>

          <!-- 分配权限对话框 -->
        <el-dialog
            title="分配权限"
            :visible.sync="setRightDialogVisible"
            width="30%"
            @close="setRightDialogClosed">
            <!-- 树形控件 -->
            <el-tree :data="rightsList"
            :props="treeProps"
            show-checkbox
            default-expand-all
            :default-checked-keys="defkeys"
            node-key="id"
            ref="treeRef"
             >
             </el-tree>

            <!-- 底部区域 -->
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="allotRights">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>
<script>
export default {
  data () {
    return {
      // 添加角色表单验证规则
      addRoleRules: {
        roleName: [
          { required: true, message: '请输入角色名', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在3 到 10个字符', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10个字符', trigger: 'blur' }
        ]
      },
      // 所有角色列表数据
      rolesList: [],
      // 添加角色表单数据
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      //   编辑角色表单数据
      editForm: {},
      // 控制添加角色表单对话框的显示与隐藏
      addDialogVisible: false,
      // 控制编辑角色表单对话框的显示与隐藏
      editDialogVisible: false,
      // 控制修改权限对话框的显示与隐藏
      setRightDialogVisible: false,
      // 所有权限数据列表
      rightsList: [],
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 这是默认选中的节点ID值数组
      defkeys: [],
      roleId: ''
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    // 获取所有角色列表
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取角色列表失败')
      this.rolesList = res.data
    },
    // 添加角色
    async addRole () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('roles', this.addForm)
        if (res.meta.status !== 200) this.$message.error('添加失败')
        this.$message.success('添加成功')
        this.addDialogVisible = false
        this.getRolesList()
      })
    },
    async delRole (id) {
      const confirmResult = await this.$confirm('此操作将删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 如果confirmResult等于confirm表示用户点击了确认，等于cancle表示用户点击了取消
      if (confirmResult !== 'confirm') return this.$message.error('已取消删除')
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除失败')
      this.$message.success('删除成功')
      this.getRolesList()
    },
    // 关闭添加对话框时重置内容
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    // 关闭编辑对话框时重置内容
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    // 展示编辑角色对话框
    async showEditRoleForm (id) {
      const { data: res } = await this.$http.get('roles/' + id)
      console.log(res)
      if (res.meta.status !== 200) return this.$message.error('获取角色失败')
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 编辑角色
    async editRoleForm () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        // 发起修改用户信息的数据请求
        const { data: res } = await this.$http.put('roles/' + this.editForm.roleId, { roleName: this.editForm.roleName,
          roleDesc: this.editForm.roleDesc })
        if (res.meta.status !== 200) return this.$message.error('更新失败')
        // 关闭对话框，刷新列表，提示成功
        this.editDialogVisible = false
        this.getRolesList()
        this.$message.success('更新用户成功')
      })
    },
    // 根据id删除对应的权限
    async removeRightById (role, rightId) {
      // 根据弹框提示用户是否要删除
      const confirmResult = await this.$confirm('此操作将删除该权限,是否继续', '提示', {
        confirmButtonText: '确定',
        cancleButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') return this.$message.info('取消删除')
      this.$message.info('确认了删除')

      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) return this.$message.error('删除权限失败')
      role.children = res.data
    },
    // 展示分配权限对话框
    async showSetRightDialog (role) {
      // 获取所有权限数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('获取权限数据失败')
      this.rightsList = res.data
      // 递归获取三级节点的id并且赋给defkeys
      this.getLeafKeys(role, this.defkeys)
      this.setRightDialogVisible = true
      // 保存id到数据中供后续使用
      this.roleId = role.id
    },
    // 通过递归的形式，获取所有权限的ID并且保存到defkeys数组中
    getLeafKeys (node, arr) {
      // 如果当前node节点不包含children属性则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed () {
      this.defkeys = []
    },
    // 分配权限
    async allotRights () {
      const keys = [...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()]
      console.log(keys)
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) return this.$message.error('分配权限失败 ')
      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
    margin: 7px
}

.bdtop {
    border-top: 1px solid #eee
}

.bdbottom {
    border-bottom: 1px solid #eee;
}

.vcenter {
    display: flex;
    align-items: center;
}
</style>
