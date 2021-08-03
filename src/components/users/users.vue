<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区 -->
    <el-card class="box-card">
      <el-row :gutter="20" type="flex" justify="space-between">
        <el-col :span="10">
          <el-input
            placeholder="请输入内容"
            class="input-with-select"
            v-model="queryInfo.query"
            clearable
            @clear="clearInput"
          >
            <template #append>
              <el-button icon="el-icon-search" @click="searchQuery"></el-button>
            </template> </el-input
        ></el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogOpen">添加用户</el-button>
        </el-col>
      </el-row>
      <el-table :data="tableData" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="username" label="用户名" width="120">
        </el-table-column>
        <el-table-column prop="role_name" label="角色" width="120">
        </el-table-column>
        <el-table-column prop="email" label="邮箱"> </el-table-column>
        <el-table-column prop="mobile" label="电话"> </el-table-column>
        <el-table-column label="状态" width="70">
          <template #default="scope">
            <el-switch
              v-model="scope.row.mg_state"
              active-color="#13ce66"
              inactive-color="#ff4949"
              @change="stateChange(scope.row)"
            ></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="170">
          <template #default="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              circle
              @click="editDialogOpen(scope.row.id)"
            ></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              circle
              @click="handleDelete(scope.$index, scope.row)"
            ></el-button>
            <el-tooltip
              class="item"
              effect="dark"
              content="分配角色"
              placement="top-start"
              :enterable="false"
            >
              <el-button
                type="warning"
                icon="el-icon-star-off"
                circle
                @click="openRoleDialog(scope.row)"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页插件 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :page-sizes="[2, 4, 6, 8]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
      <!-- 添加用户对话框 -->
      <el-dialog
        title="添加用户"
        :visible.sync="addDialogVisible"
        width="50%"
        @close="addDialogClosed"
      >
        <el-form
          :model="addForm"
          :rules="addFormRules"
          label-width="70px"
          ref="addFormRef"
        >
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addForm.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="addForm.password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机号" prop="mobile">
            <el-input v-model="addForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <template #footer>
          <span class="dialog-footer">
            <el-button @click="addDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addUser">确 定</el-button>
          </span>
        </template>
      </el-dialog>
      <!-- 修改用户对话框 -->
      <el-dialog title="添加用户" :visible.sync="editDialogVisible" width="50%">
        <el-form
          :model="editForm"
          :rules="editFormRules"
          label-width="70px"
          ref="addFormRef"
        >
          <el-form-item label="用户名">
            <el-input v-model="editForm.username" disabled></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机号" prop="mobile">
            <el-input v-model="editForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <template #footer>
          <span class="dialog-footer">
            <el-button @click="editDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editUser">确 定</el-button>
          </span>
        </template>
      </el-dialog>
    </el-card>
    <!-- 分配角色对话框 -->
    <el-dialog title="分配角色" :visible.sync="roleDialogVisible" width="40%">
      <div>
        <p>当前用户名：{{ userinfo.username }}</p>
        <p>当前用户角色：{{ userinfo.role_name }}</p>
        <p>
          请选择角色:
          <el-select v-model="selectedRoleId" placeholder="请选择" multiple>
            <el-option
              v-for="item in rolesList"
              :key="item.id"
              :label="item.roleName"
              :value="item.id"
            >
            </el-option>
          </el-select>
        </p>
      </div>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="closeSetRoleDialog">取 消</el-button>
          <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      roleDialogVisible: false,
      userinfo: {},
      rolesList: [],
      selectedRoleId: '',
      queryInfo: {
        query: '',
        // 当前页的页数
        pagenum: 1,
        // 每页显示多少页
        pagesize: 2
      },
      //   数据总条数
      total: 0,
      tableData: [],
      haha: true,
      //   控制添加用户对话框的显示与否
      addDialogVisible: false,
      //   控制修改用户对话框的显示与否
      editDialogVisible: false,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      editForm: {
        username: '',
        email: '',
        mobile: ''
      },
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '用户名在3到10个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 10, message: '密码在3到10个字符', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { type: 'email', message: '请输入有效的邮箱类型', trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          {
            type: 'string',
            pattern: /^1[0-9]{10}$/,
            message: '请输入有效的手机号',
            trigger: 'blur'
          }
        ]
      },
      editFormRules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { type: 'email', message: '请输入有效的邮箱类型', trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          {
            type: 'string',
            pattern: /^1[0-9]{10}$/,
            message: '请输入有效的手机号',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  components: {},
  methods: {
    async getUserList () {
      const { data: res } = await this.$http.get('/users', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取用户列表失败')
      }
      this.tableData = res.data.users
      //   总条数
      this.total = res.data.total
    },
    // 编辑用户
    handleEdit (index, row) {
      console.log(index, row)
    },
    // 删除用户
    handleDelete (index, row) {
      this.$confirm('是否确定删除该用户?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          const { data: res } = await this.$http.delete('/users/' + row.id)
          if (res.meta.status !== 200) {
            return this.$message.error('删除用户失败')
          }
          //   刷新用户信息
          this.getUserList()
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
    },
    // 每页显示条数改变时触发
    handleSizeChange (val) {
      //   console.log(`每页 ${val} 条`)
      this.queryInfo.pagesize = val
      //   发送请求获取数据
      this.getUserList()
    },
    // 当前页改变时触发
    handleCurrentChange (val) {
      this.queryInfo.pagenum = val
      this.getUserList()
    },
    // switch的状态改变时触发
    async stateChange (row) {
      console.log(row)
      const { data: res } = await this.$http.put(
        `users/${row.id}/state/${row.mg_state}`
      )
      if (res.meta.status !== 200) {
        //   更新失败时将状态复原
        row.mg_state = !row.mg_state
        return this.$message.error('更新用户状态失败')
      }
      this.$message.success('更新用户状态成功')
    },
    // 条件查询
    searchQuery () {
      this.queryInfo.pagenum = 1
      this.getUserList()
    },
    clearInput () {
      this.queryInfo.pagenum = 1
      this.getUserList()
    },
    addUser () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('请填写正确的用户信息')
        } else {
          // 发起添加用户的请求
          const { data: res } = await this.$http.post('/users', this.addForm)
          if (res.meta.status !== 201) {
            this.$message.error('添加用户失败')
          } else {
            this.$message.success('添加用户成功')
            this.addDialogVisible = false
            // 刷新列表
            this.getUserList()
          }
        }
      })
    },
    // 打开添加用户对话框
    addDialogOpen () {
      this.addDialogVisible = true
    },
    async editDialogOpen (id) {
      this.editDialogVisible = true
      const { data: res } = await this.$http.get('/users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户信息失败，无法修改')
      }
      this.editForm = res.data
    },
    async addDialogClosed () {
      // 关闭表单时清空数据
      this.$refs.addFormRef.resetFields()
    },
    // 修改用户
    editUser () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('请填写正确的用户信息')
        } else {
          // 发起添加用户的请求
          const { data: res } = await this.$http.put(
            '/users/' + this.editForm.id,
            this.editForm
          )
          if (res.meta.status !== 200) {
            this.$message.error('添加用户失败')
          } else {
            this.$message.success('添加用户成功')
            this.editDialogVisible = false
            // 刷新列表
            this.getUserList()
          }
        }
      })
    },
    // 打开用户分配角色对话框
    async openRoleDialog (row) {
      this.userinfo = row
      // 查询所有角色
      const { data: res } = await this.$http.get('/roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolesList = res.data
      this.roleDialogVisible = true
    },
    closeSetRoleDialog () {
      this.selectedRoleId = ''
      this.roleDialogVisible = false
    },
    // 保存用户角色
    async saveRoleInfo () {
      if (!this.selectedRoleId) {
        return this.$message.warn('请选择要分配的角色')
      }
      const { data: res } = await this.$http.put(
        `users/${this.userinfo.id}/role`,
        {
          rid: this.selectedRoleId
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('更新用户角色失败')
      }
      this.getUserList()
      this.roleDialogVisible = false
    }
  },
  created () {
    this.getUserList()
  }
}
</script>

<style lang="less" scoped>
.el-table {
  margin-top: 10px;
  font-size: 12px;
}
</style>
