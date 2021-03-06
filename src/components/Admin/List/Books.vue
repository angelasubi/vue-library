<template>
  <div>
    <div class="header">
      <h3>用户信息列表</h3>
      <div class="create">
        <el-button type="success" @click="createUser">新增</el-button>
      </div>
    </div>

    <div class="content">
      <el-form :inline="true" :model="form" class="demo-form-inline">
        <el-form-item label="姓名">
          <el-input v-model="form.name" placeholder="请输入姓名"></el-input>
        </el-form-item>
        <el-form-item label="手机号">
          <el-input v-model="form.mobile" placeholder="请输入手机号"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="getData">查询</el-button>
        </el-form-item>
      </el-form>

      <el-table :data="tableData" border stripe class="e-table">
        <el-table-column type="index" label="序号" width="80"></el-table-column>
        <el-table-column label="姓名" prop="name" width="80"></el-table-column>
        <el-table-column label="性别" prop="gender" width="80"></el-table-column>
        <el-table-column label="年龄" width="80">
          <template slot-scope="scope">
            {{ age(scope.row.birthday) }}
          </template>
        </el-table-column>
        <el-table-column label="出生日期">
          <template slot-scope="scope">
            {{ time(scope.row.birthday) }}
          </template>
        </el-table-column>
        <el-table-column label="手机号" prop="mobile"></el-table-column>
        <el-table-column label="邮箱" prop="email"></el-table-column>
        <el-table-column label="地址" prop="address"></el-table-column>
        <el-table-column label="操作" width="200">
          <template slot-scope="scope">
            <el-button type="text" @click="editUserData(scope.row)">修改</el-button>
            <el-button type="text" @click="deleteUserData(scope.row)">删除</el-button>
            <el-button type="text" @click="borroweMessage(scope.row)">借阅信息</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <delete-dialog :getDeleteId="deleteId"></delete-dialog>

    <div class="book-pagination">
      <el-pagination
        :current-page="page"
        :page-size="perPage"
        :page-sizes="[1, 20, 40, 100, 200]"
        :total="total"
        @current-change="handleCurrentChange"
        @size-change="handleSizeChange"
        layout="total, sizes, prev, pager, next, jumper">
      </el-pagination>
    </div>
  </div>
</template>

<script>
import storage from 'common/js/store'
import formatDate from 'common/js/date'
import DeleteDialog from './DeleteDialog.vue'
import {mapState} from 'vuex'

export default {
  components: {
    DeleteDialog
  },

  data() {
    return {
      nowTime: null,
      form: {
        name: '',
        mobile: ''
      },
      tableData: [],
      page: 1,
      perPage: 10,
      total: null,
      deleteId: ''
    }
  },
  computed: {
    ...mapState(['dialog_visible'])
  },

  watch: {
    dialog_visible(status) {
      if (!status) {
        this.getData()
      }
    }
  },

  methods: {
    getToken() {
      this.nowTime = new Date().getTime()
      if (storage().get('admin_token') == null) {
        this.$message({type: 'error', message: '管理员信息过期，请重新登录'})
        this.$router.push({path: '/admin/login'})
      } else {
        if (this.nowTime - JSON.parse(storage().get('admin_token')).time > 3600000) {
          this.$message({type: 'error', message: '管理员信息过期，请重新登录'})
          this.$router.push({path: '/admin/login'})
        }
      }
    },

    getData() {
      this.$http.get('admin/list',{
        params: {
          page: this.page,
          per_page: this.perPage,
          name: this.form.name,
          mobile: this.form.mobile
        }
      })
      .then(resp => {
        this.tableData = resp.data.data
        this.total = resp.data.total
        this.per_page = +resp.data.per_page
        this.page = resp.data.current_page
      })
    },

    deleteUserData(scope) {
      this.deleteId = scope._id
      this.$store.dispatch('dialog_visible')
    },

    borroweMessage(scope) {
      this.$router.push({name: 'admin-message', query: {id: scope._id}})
    },

    createUser() {
      this.$store.dispatch('user_data', 1)
      this.$router.push('operate')
    },
    editUserData(scope) {
      this.$store.dispatch('user_data', scope)
      this.$router.push('operate')
    },

    handleCurrentChange(page) {
      this.page = page
      this.getData()
    },
    handleSizeChange(perPage) {
      this.perPage = perPage
      this.getData()
    },
    time(t) { return formatDate(t) },
    age(t) { return formatDate(new Date()).substr(0, 4) - formatDate(t).substr(0, 4) }
  },

  created() {
    this.getToken()
    this.getData()
  }
}
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
.header
  width: 1200px
  height: 64px
  margin: 0 auto
  overflow: hidden
  h3
    float: left
  .create
    float: right
    height: 64px
    line-height: 64px
.content
  width: 1200px
  margin: 0 auto
.book-pagination
  width: 1200px
  margin: 20px auto
  overflow: hidden
  .el-pagination
    float: right
</style>
