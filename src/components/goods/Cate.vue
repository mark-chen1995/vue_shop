<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区 -->
    <el-card class="box-card">
      <el-button type="primary" @click="addCateDialog">添加分类</el-button>
      <tree-table
        :data="tableData"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
      >
        <template #valid="scope">
          <i
            class="el-icon-success"
            style="color: green !important;"
            v-if="scope.row.cat_deleted === false"
          ></i>
          <i class="el-icon-error" style="color: red !important;" v-else></i>
        </template>
        <template #order="scope">
          <el-tag type="success" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag type="info" v-else-if="scope.row.cat_level === 1"
            >二级</el-tag
          >
          <el-tag type="warning" v-else>三级</el-tag>
        </template>
        <template #operator>
          <el-button type="primary" size="mini">
            <i class="el-icon-edit"></i>
            <span>编辑</span>
          </el-button>
          <el-button type="danger" size="mini">
            <i class="el-icon-delete"></i>
            <span>删除</span>
          </el-button>
        </template>
      </tree-table>

      <!-- 分页插件 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :page-size="queryInfo.pagesize"
        layout="total, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
      <!-- 添加分类对话框 -->
      <el-dialog
        title="添加分类"
        :visible.sync="cateDialogVisible"
        width="40%"
        @click="addCateDialogClosed"
        ref="cateDialog"
        @close="addCateDialogClosed"
      >
        <el-form
          ref="cateForm"
          :model="cateForm"
          label-width="80px"
          :rules="cateFormRules"
        >
          <el-form-item label="分类名称" prop="cat_name">
            <el-input v-model="cateForm.cat_name"></el-input>
          </el-form-item>
          <el-form-item label="父级别分类" label-width="90px">
            <el-cascader
              ref="cateCascader"
              clearable
              v-model="selectedKeys"
              :options="parentCateList"
              :props="cascadeProps"
            ></el-cascader>
          </el-form-item>
        </el-form>
        <template #footer>
          <span class="dialog-footer">
            <el-button @click="closeCateDialog">取 消</el-button>
            <el-button type="primary" @click="saveCateInfo">确 定</el-button>
          </span>
        </template>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 分类数据
      tableData: [],
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      //   添加分类的对话框是否显示
      cateDialogVisible: false,
      //   为tree-table指定列定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'valid'
        },
        {
          label: '排序',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'operator'
        }
      ],
      cateForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      cateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', triger: 'blur' }
        ]
      },
      //   父级分类列表
      parentCateList: [],
      //   指定级联选择器的属性
      cascadeProps: {
        expandTrigger: 'click',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: true
      },
      //   选中的父类分类的id数组
      selectedKeys: []
    }
  },
  components: {},
  methods: {
    //   获取全部分类信息
    async getCateList () {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }
      this.tableData = res.data.result
      this.total = res.data.total
    },
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 打开添加分类对话框
    async addCateDialog () {
      this.cateDialogVisible = true
      //   调用api获取所有一二级分类
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类信息失败')
      }
      this.parentCateList = res.data
    },
    // 关闭添加分类对话框
    closeCateDialog () {
      this.cateDialogVisible = false
    },
    saveCateInfo () {
      // 如果选中父级分类，则需要修改要提交的cateForm值
      if (this.selectedKeys.length > 0) {
        this.cateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.cateForm.cat_level = this.selectedKeys.length
      }
      //   数据检验通过后调用api保存数据
      this.$refs.cateForm.validate(async valid => {
        if (!valid) {
          return this.$message.error('请填写分类名')
        } else {
          // 发起添加用户的请求
          const { data: res } = await this.$http.post(
            'categories',
            this.cateForm
          )
          if (res.meta.status !== 201) {
            return this.$message.error('添加分类失败')
          }
          // 刷新列表
          this.getCateList()
          this.cateDialogVisible = false
          console.log(this.cateDialogVisible)
          this.$message.success('添加分类成功')
        }
      })
    },
    // 关闭对话框时重置数据
    addCateDialogClosed () {
      this.$refs.cateForm.resetFields()
      this.selectedKeys = []
      this.cateForm.cat_pid = 0
      this.cateForm.cat_level = 0
    }
  },
  created () {
    this.getCateList()
  }
}
</script>

<style lang="less" scoped>
.el-card__body .el-button {
  margin-bottom: 15px;
}
.el-cascader {
  width: 100%;
}
</style>
