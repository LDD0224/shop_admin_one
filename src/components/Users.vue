<template>
  <div class="users">
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索栏 -->
    <el-input placeholder="请输入内容" v-model="query" class="input-with-select">
      <el-button slot="append" icon="el-icon-search" @click="search"></el-button>
    </el-input>
    <el-button type="success" plain @click="showAddDialog">添加用户</el-button>
    <!-- 表格组件 -->
    <!--
      el-table : 表格组件
      data : 指定表格最终需要渲染的数据（数组）
      style： 让表格宽度100%
      el-table-column： 定义表格的每一列
        label="日期"： 列的标题
        prop： 对应显示的数据的属性名
        width： 列的宽度 不支持百分比

      在el-tabel中，如果想要自定义列模版
      在el-table-column中使用template
      <template slot-scope="scope">自己定义的内容</template>
    -->
    <el-table :data="userList" style="width: 100%">
      <el-table-column prop="username" label="姓名" width="180"></el-table-column>
      <el-table-column prop="email" label="邮箱" width="180"></el-table-column>
      <el-table-column prop="mobile" label="电话"></el-table-column>
      <el-table-column prop="mg_state" label="用户状态">
        <!-- 在自定义列模版中，如何访问到当前列的数据 -->
        <template slot-scope="scope">
          <el-switch v-model="scope.row.mg_state" active-color="#13ce66" inactive-color="#ff4949"></el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" plain size="mini"></el-button>
          <el-button type="danger" icon="el-icon-delete" @click="delUser(scope.row.id)" plain size="mini"></el-button>
          <el-button type="success" icon="el-icon-check" plain size="mini">分配角色</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!--
      分页
      @size-change: 表示每页的条数发生了改变
      @current-change: 当前页发生改变
      current-page: 指定当前页面
      page-sizes: 指定选择每页条数的数组
      page-size: 每页的条数
      total:指定总条数
      layout: 指定分页的空间 控制分页显示的顺序
    -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage"
      :page-sizes="[2, 4, 6, 8]"
      :page-size="pageSize"
      :total="total"
      background
      layout="total, sizes, prev, pager, next, jumper"
      >
    </el-pagination>
  </div>
</template>

<script>
// 引入axios
// import axios from 'axios'
export default {
  data() {
    return {
      // 用户列表
      userList: [],
      query: '',
      currentPage: 1,
      pageSize: 2,
      total: 0
    }
  },
  methods: {
    getUserList() {
      // axios如果是get/delete请求，参数要么直接拼地址栏，要么放到params中
      // 如果post/put/patch请求，参数放到data中
      // 除了login请求，其他所有的接口都必须携带token， 要求设置给请求头：Authorization
      this.axios({
        method: 'get',
        url: 'users',
        params: {
          query: this.query,
          pagenum: this.currentPage,
          pagesize: this.pageSize
        }
      }).then(res => {
        let {
          meta: { status },
          data: { users, total }
        } = res
        if (status === 200) {
          this.userList = users
          this.total = total
        }
      })
    },
    handleSizeChange(val) {
      // 修改this.pageSize
      this.pageSize = val
      this.getUserList()
    },
    handleCurrentChange(val) {
      this.currentPage = val
      this.getUserList()
    },
    search() {
      // 搜索的时候，把当前页第一页
      this.currentPage = 1
      this.getUserList()
    },
    delUser(id) {
      this.$confirm('你确定要删除吗', '温馨提示', {
        type: 'wraning'
      })
        .then(() => {
          // 发送ajax请求，删除数据
          return this.axios({
            method: 'delete',
            url: `users/${id}`
          })
        })
        .then(res => {
          if (res.meta.status === 200) {
            // 如果我们发现当前页只有一条数据了，应该current减1，渲染上一页
            if (this.userList.length <= 1 && this.currentPage > 1) {
              this.currentPage--
            }
            this.getUserList()
            this.$message.success('删除成功了')
          }
        })
        .catch(() => {
          this.$message.info('删除取消了')
        })
    },
    // 显示添加对话框
    showAddDialog() {
      this.showAddDialog = true
    }
  },
  created() {
    this.getUserList()
  }
}
</script>

<style lang="less" scoped>
.el-breadcrumb {
  height: 50px;
  line-height: 50px;
  margin-bottom: 5px;
  background-color: #d3d4dd;
}
.input-with-select {
  width: 300px;
  margin-bottom: 5px;
}
</style>
