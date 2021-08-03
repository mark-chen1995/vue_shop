<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <el-table :data="rolesList" border stripe>
        <!-- 可展开列 -->
        <el-table-column type="expand">
          <template #default="props">
            <el-row
              v-for="(item, index) in props.row.children"
              :key="item.id"
              :class="['vcenter', 'bdbuttom', index === 0 ? 'bdtop' : '']"
            >
              <!-- 一级权限 -->
              <el-col :span="6"
                ><el-tag closable>{{ item.authName }}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二、三级权限 -->
              <el-col :span="18">
                <!-- 二级权限 -->
                <el-row
                  v-for="(item1, index1) in item.children"
                  :key="item1.id"
                  :class="['vcenter', index1 === 0 ? '' : 'bdtop']"
                >
                  <el-col :span="6"
                    ><el-tag
                      type="success"
                      @close="removeRightById(props.row, item1.id)"
                      closable
                      >{{ item1.authName }}</el-tag
                    ><i class="el-icon-caret-right"></i
                  ></el-col>
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      closable
                      v-for="item2 in item1.children"
                      :key="item2.id"
                      @close="removeRightById(props.row, item2.id)"
                      >{{ item2.authName }}</el-tag
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column label="操作" width="320px">
          <template #default="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit"
              >编辑</el-button
            >
            <el-button size="mini" type="danger" icon="el-icon-delete"
              >删除</el-button
            >
            <el-button
              size="mini"
              type="warning"
              icon="el-icon-star-off"
              @click="openSetRightDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 权限设置对话框 -->
    <el-dialog
      title="权限设置"
      :visible.sync="setRightDialogVisible"
      width="40%"
    >
      <el-tree
        :data="rightsList"
        :props="rightTree"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defKeys"
        @close="closeSetRightDialog"
        ref="rightTree"
      ></el-tree>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="closeSetRightDialog">取 消</el-button>
          <el-button type="primary" @click="allowRights">确 定</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rolesList: [],
      setRightDialogVisible: false,
      rightsList: [],
      rightTree: {
        label: 'authName',
        children: 'children'
      },
      defKeys: [],
      rowId: -1
    }
  },
  components: {},
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('/roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败')
      }
      this.rolesList = res.data
    },
    // 根据id删除权限
    removeRightById (row, id) {
      this.$confirm('是否确定删除该权限?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          const { data: res } = await this.$http.delete(
            `/roles/${row.id}/rights/${id}`
          )
          if (res.meta.status !== 200) {
            return this.$message.error('无法删除权限')
          }
          //   更新页面，但这会导致整个页面重新刷新
          //   this.getRolesList()
          // 通过返回结果只要更新绑定的数据
          row.children = res.data
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
    async openSetRightDialog (row) {
      this.rowId = row.id
      // 获取角色所有权限信息
      const { data: res } = await this.$http.get('/rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败')
      }
      this.rightsList = res.data
      // 将三级权限id值赋给defKeys，在树形控件中使用该属性值确定选中项
      this.getAllRights(row, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 递归获取对象的全部三级权限，并赋值给数组
    getAllRights (data, arr) {
      if (data.children === undefined) {
        return arr.push(data.id)
      }
      data.children.forEach(element => {
        this.getAllRights(element, arr)
      })
    },
    async allowRights () {
      const checkedKesy = [
        ...this.$refs.rightTree.getCheckedKeys(),
        ...this.$refs.rightTree.getHalfCheckedKeys()
      ]
      const keyStr = checkedKesy.join(',')
      // 调用api保存权限信息
      const { data: res } = await this.$http.post(
        `roles/${this.rowId}/rights`,
        {
          rids: keyStr
        }
      )
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('保存权限失败')
      }
      this.$message.success('保存权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    },
    closeSetRightDialog () {
      this.setRightDialogVisible = false
      // 关闭对话框时清空defkeys数组
      this.defKeys = []
    }
  },
  created () {
    this.getRolesList()
  }
}
</script>

<style lang="less" scoped>
.el-button {
  margin-bottom: 15px;
}
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid;
}
.bdbuttom {
  border-bottom: 1px solid;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
