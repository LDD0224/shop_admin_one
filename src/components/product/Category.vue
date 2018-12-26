<template>
  <div class="category">
    <el-button type="success" @click="showAddDialog" plain>添加商品分类</el-button>
    <!-- 表格组件显示权限数据 -->
    <el-table :data="categoryList">
      <!--
        tree-key: 嵌套解析的key 默认id
        childKey: 查找子属性的属性名  默认是children
        parentKey: 指定父节点的id值  如果不设置，那么节点找不到爹，没办法缩起来
        levelKey: 指定节点的深度，级别
        indent-size:指定缩进
      -->
      <!-- 每一列 -->
      <el-table-tree-column label="分类名称" prop="cat_name" tree-key="cat_id" parent-key="cat_pid" level-key="cat_level" :indentSize="20"></el-table-tree-column>
      <el-table-column label="是否删除" prop="cat_deleted">
        <template slot-scope="{row}">{{row.cat_deleted ? '是' : '否'}}</template>
      </el-table-column>
      <el-table-column label="排序" prop="cat_level"></el-table-column>
      <el-table-column label="操作">
        <template slot-scope="{row}">
          <el-button type="primary" icon="el-icon-edit" plain size="mini" ></el-button>
          <el-button type="danger" icon="el-icon-delete" plain size="mini" @click="delCategory(row)"></el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <!--
      @size-change: 表示每页的条数发生了改变，会触发handleSizeChange
      @current-change: 当前页发生改变
      current-page: 指定当前页面
      page-sizes: 指定选择每页条数的数组
      page-size: 每页的条数
      total:指定总条数
      layout: 指定分页的空间
    -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage"
      :page-sizes="[10, 20, 30, 40]"
      :page-size="pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
      background
    ></el-pagination>
    <!-- 添加表单 -->
    <el-dialog title="添加分类" :visible.sync="addDialogVisible" width="40%">
      <el-form ref="addForm" :model="addForm" label-width="80px" :rules="rules" status-icon>
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addForm.cat_name" placeholder="请输入分类名称"></el-input>
        </el-form-item>
        <!-- cat_pid必须是一个数组 -->
        <el-form-item label="父级名称" prop="cat_pid">
          <el-cascader :options="options" v-model="addForm.cat_pid" change-on-select clearable :props="props"></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCategory">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      categoryList: [],
      currentPage: 1,
      pageSize: 10,
      total: 0,
      addDialogVisible: false,
      addForm: {
        cat_id: [],
        cat_name: '',
        cat_level: ''
      },
      options: [],
      props: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      rules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    // 获取商品分类列表
    async getCategoryList() {
      let res = await this.axios.get('categories', {
        params: {
          type: 3,
          pagenum: this.currentPage,
          pagesize: this.pageSize
        }
      })
      let {
        meta: { status },
        data: { result, total }
      } = res
      if (status === 200) {
        this.total = total
        this.categoryList = result
        // console.log(this.categoryList)
      }
    },
    // 修改了每页的条数
    handleSizeChange(val) {
      this.pageSize = val
      this.currentPage = 1
      this.getCategoryList()
    },
    // 修改了当前页的页码
    handleCurrentChange(val) {
      this.currentPage = val
      this.getCategoryList()
    },
    async showAddDialog() {
      this.addDialogVisible = true
      // 加载商品的分类数据（2级）
      let res = await this.axios.get('categories?type=2')
      if (res.meta.status === 200) {
        this.options = res.data
      }
    },
    addCategory() {
      this.$refs.addForm.validate(async valid => {
        if (!valid) return false
        // 发送ajax请求  cat_name  cat_pid  cat_level
        let { cat_pid: catPid, cat_name: catName } = this.addForm
        let res = await this.axios.post('categories', {
          cat_pid: catPid[catPid.length - 1] || 0,
          cat_name: catName,
          cat_level: catPid.length
        })
        if (res.meta.status === 201) {
          this.$refs.addForm.resetFields()
          this.addDialogVisible = false
          this.getCategoryList()
          this.$message.success('添加成功了')
        } else {
          this.$message.info('添加取消了')
        }
      })
    },
    // 删除商品分类
    async delCategory(row) {
      try {
        await this.$confirm('你确定要删除吗', '温馨提示', {
          type: 'warning'
        })
        // 发送ajax请求，删除数据
        let res = await this.axios.delete(`categories/${row.cat_id}`)
        if (res.meta.status === 200) {
          this.getCategoryList()
          this.$message.success('删除成功了')
        }
      } catch (e) {
        this.$message.info('删除取消了')
      }
    }
  },
  created() {
    this.getCategoryList()
  }
}
</script>

<style>

</style>
