<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区 -->
    <el-card>
      <el-alert
        title="注意：只允许为第三级分类设计相关参数"
        type="warning"
        show-icon
        :closable="false"
      ></el-alert>
      <div>
        <span>选择商品分类：</span>
        <el-cascader
          ref="cateCascader"
          clearable
          v-model="selectedKeys"
          :options="cateList"
          :props="cascadeProps"
          size="medium"
          @change="selectionChange"
        ></el-cascader>
      </div>
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <el-tab-pane label="动态参数" name="many" :disabled="isBtnDisable">
          <el-button type="primary" :disabled="isBtnDisable" @click="openDialog"
            >添加参数</el-button
          >
          <el-table
            :data="manyTableData"
            style="width: 100%"
            :disabled="isBtnDisable"
            border
            stripe
          >
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template #default="scope">
                <el-tag
                  closable
                  type="success"
                  v-for="(item, index) in scope.row.attr_vals"
                  :key="index"
                  @close="handleClose(index, scope.row)"
                  >{{ item }}</el-tag
                >
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ 新增标签</el-button
                >
              </template>
            </el-table-column>
            <el-table-column label="#" type="index"> </el-table-column>
            <el-table-column prop="attr_name" label="参数名称">
            </el-table-column>
            <el-table-column label="操作">
              <template #default="sc">
                <el-button
                  type="primary"
                  size="mini"
                  @click="showEditDialog(sc.row)"
                >
                  <i class="el-icon-edit"></i>
                  <span>编辑</span>
                </el-button>
                <el-button
                  type="danger"
                  size="mini"
                  @click="deleteParams(sc.row.attr_id)"
                >
                  <i class="el-icon-delete"></i>
                  <span>删除</span>
                </el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 静态属性页 -->
        <el-tab-pane label="静态属性" name="only" :disabled="isBtnDisable">
          <el-button type="primary" :disabled="isBtnDisable" @click="openDialog"
            >添加属性</el-button
          >
          <el-table
            :data="onlyTableData"
            style="width: 100%"
            :disabled="isBtnDisable"
            border
            stripe
          >
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template #default="scope">
                <el-tag
                  closable
                  type="success"
                  v-for="(item, index) in scope.row.attr_vals"
                  :key="index"
                  @close="handleClose(index, scope.row)"
                  >{{ item }}</el-tag
                >
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ 新增标签</el-button
                >
              </template>
            </el-table-column>
            <el-table-column label="#" type="index"> </el-table-column>
            <el-table-column prop="attr_name" label="属性名称">
            </el-table-column>
            <el-table-column label="操作">
              <template #default="scope">
                <el-button type="primary" size="mini">
                  <i
                    class="el-icon-edit"
                    @click="showEditDialog(scope.row)"
                  ></i>
                  <span>编辑</span>
                </el-button>
                <el-button
                  type="danger"
                  size="mini"
                  @click="deleteParams(scope.row.attr_id)"
                >
                  <i class="el-icon-delete"></i>
                  <span>删除</span>
                </el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加属性或参数的对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="dialogVisible"
      width="30%"
      @close="dialogClosed"
    >
      <el-form
        :model="addForm"
        :rules="rules"
        ref="addFormRef"
        label-width="100px"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改属性或参数的对话框 -->
    <el-dialog
      :title="'修改' + titleText"
      :visible.sync="editdialogVisible"
      width="30%"
      @close="editDialogClosed"
    >
      <el-form
        :model="editForm"
        :rules="rules"
        ref="editFormRef"
        label-width="100px"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editdialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      selectedKeys: [],
      parentCateList: [],
      cascadeProps: {
        expandTrigger: 'click',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      cateList: [],
      data: [],
      //   激活的tab页
      activeName: 'many',
      manyTableData: [],
      onlyTableData: [],
      dialogVisible: false,
      addForm: {
        attr_name: ''
      },
      rules: {
        attr_name: [{ required: true, message: '请输入内容', trigger: 'blur' }]
      },
      editdialogVisible: false,
      editForm: {
        attr_name: ''
      },
      inputVisible: false
    }
  },
  components: {},
  methods: {
    //   获取全部分类信息
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }
      this.cateList = res.data
    },
    async getParamsData () {
      const { data: res } = await this.$http.get(
        `categories/${
          this.selectedKeys[this.selectedKeys.length - 1]
        }/attributes`,
        { params: { sel: this.activeName } }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品参数失败')
      }
      res.data.forEach(element => {
        element.attr_vals = element.attr_vals
          ? element.attr_vals.split(' ')
          : []
        //   控制展开行中文本框的显示与隐藏
        element.inputVisible = false
        //   保存文本框中输入的值
        element.inputValue = ''
      })
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    // 当选择分类变化时
    selectionChange () {
      // 默认为动态参数
      this.activeName = 'many'
      this.getParamsData()
    },
    // 当tab被点击时
    handleTabClick () {
      this.getParamsData()
    },
    openDialog () {
      this.dialogVisible = true
    },
    // 关闭对话框
    dialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    // 打开对话框
    async showEditDialog (row) {
      const { data: res } = await this.$http.get(
        `categories/${
          this.selectedKeys[this.selectedKeys.length - 1]
        }/attributes/${row.attr_id}`,
        { params: { attr_sel: this.activeName } }
      )
      if (res.meta.status !== 200) {
        this.$message.error('获取参数失败')
      } else {
        this.editForm = res.data
        this.editdialogVisible = true
      }
    },
    // 添加参数
    addParams () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error(`请填写${this.titleText}`)
        } else {
          // 发起添加用户的请求
          const { data: res } = await this.$http.post(
            `categories/${
              this.selectedKeys[this.selectedKeys.length - 1]
            }/attributes`,
            { attr_name: this.addForm.attr_name, attr_sel: this.activeName }
          )
          if (res.meta.status !== 201) {
            this.$message.error(`添加${this.titleText}失败`)
          } else {
            this.$message.success(`添加${this.titleText}成功`)
            this.dialogVisible = false
            // 刷新列表
            this.getParamsData()
          }
        }
      })
    },
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    // 修改参数
    editParams () {
      console.log('log')
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error(`请填写${this.titleText}`)
        } else {
          // 发起添加用户的请求
          const { data: res } = await this.$http.put(
            `categories/${
              this.selectedKeys[this.selectedKeys.length - 1]
            }/attributes/${this.editForm.attr_id}`,
            { attr_name: this.editForm.attr_name, attr_sel: this.activeName }
          )
          if (res.meta.status !== 200) {
            this.$message.error(`修改${this.titleText}失败`)
          } else {
            this.$message.success(`修改${this.titleText}成功`)
            // 刷新列表
            this.getParamsData()
            this.editdialogVisible = false
          }
        }
      })
    },
    // 删除参数
    async deleteParams (id) {
      console.log(id)
      const { data: res } = await this.$http.delete(
        `categories/${
          this.selectedKeys[this.selectedKeys.length - 1]
        }/attributes/${id}`
      )
      if (res.meta.status !== 200) {
        this.$message.error(`删除${this.titleText}失败`)
      } else {
        this.$message.success(`删除${this.titleText}成功`)
        // 刷新列表
        this.getParamsData()
      }
    },
    showInput (row) {
      row.inputVisible = true
      //   让文本框自动获得焦点，$nextTick表示在下一次DOM更新循环结束后执行延迟回调
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 添加新标签
    async handleInputConfirm (row) {
      if (row.inputValue.length === 0) {
        return (row.inputVisible = false)
      }
      row.attr_vals.push(row.inputValue)
      row.inputVisible = false
      row.inputValue = ''
      this.saveAttrVals(row)
    },
    // 删除参数
    handleClose (index, row) {
      row.attr_vals.splice(index, 1)
      this.saveAttrVals(row)
    },
    async saveAttrVals (row) {
      //   调用api保存
      const { data: res } = await this.$http.put(
        `categories/${
          this.selectedKeys[this.selectedKeys.length - 1]
        }/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(' ')
        }
      )
      if (res.meta.status !== 200) {
        this.$message.error('修改参数失败')
      } else {
        this.$message.success('修改参数成功')
      }
    }
  },
  computed: {
    isBtnDisable () {
      if (this.selectedKeys.length === 3) {
        return false
      }
      return true
    },
    // 对话框标题
    titleText () {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
  },
  created () {
    this.getCateList()
  }
}
</script>

<style lang="less" scoped>
.el-cascader {
  width: 250px;
  margin-top: 20px;
  margin-bottom: 20px;
}
.el-tag {
  margin-left: 10px;
  margin-right: 10px;
}
.button-new-tag {
  margin-left: 10px;
  height: 32px;
  line-height: 30px;
  padding-top: 0;
  padding-bottom: 0;
}
.input-new-tag {
  width: 90px;
  margin-left: 10px;
  vertical-align: bottom;
}
</style>
