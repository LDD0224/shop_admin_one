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
    async showAddDialog() {
      this.addDialogVisible = true
      // 加载商品的分类数据（2级）
      let res = await this.axios.get('categories?type=2')
      if (res.meta.status === 200) {
        this.options = res.data
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
