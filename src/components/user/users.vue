<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 搜索/添加 -->
      <el-row :gutter="10">
        <el-col :span="8">
          <el-input placeholder="请输入要查找的姓名" v-model="USerInfo.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" icon="el-icon-plus" @click="addDialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>

      <!-- 表格--用户列表 -->
      <el-table :data="userList" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="姓名" prop="username"></el-table-column>
        <el-table-column label="邮箱" prop="email" min-width="130px"></el-table-column>
        <el-table-column label="电话" prop="mobile" min-width="130px"></el-table-column>
        <el-table-column label="角色" prop="role_name" sortable></el-table-column>
        <el-table-column label="状态" prop="mg_state" sortable>
          <template slot-scope="scope">
            <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)"></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" min-width="210px" align="center">
          <template slot-scope="scope">
            <!-- 修改 -->
            <el-button type="primary" size="mini" @click="showEditDialog(scope.row.id)">修改</el-button>
            <!-- 删除 -->
            <el-button type="danger" size="mini" @click="removeUserById(scope.row.id)">删除</el-button>
            <!-- 分配角色按钮 -->
            <el-tooltip class="item" effect="dark" content="修改内容" placement="top" :enterable="false">
              <el-button type="info" size="mini">角色</el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页 -->
      <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage"
      :page-sizes="[5, 10, 20, 50]"
      :page-size="5"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    </el-card>

    <!-- 添加用户 -->
    <el-dialog
    title="添加用户:"
    :visible.sync="addDialogVisible"
    width="50%"
    @close="addDialogClosed">
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="80px" class="demo-ruleForm">
        <el-form-item label="用户名:" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码:" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱:" prop="email">
          <el-input v-model="addForm.email" type="email"></el-input>
        </el-form-item>
        <el-form-item label="手机号:" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改用户信息 -->
    <el-dialog
      title="修改用户信息"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="80px">
        <el-form-item label="用户名:">
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱:" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机号:" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    // 验证手机号的规则
    var cheackMobile = (rule, value, cb) => {
      const regMobile = /^(13[0-9]|14[01456879]|15[0-35-9]|16[2567]|17[0-8]|18[0-9]|19[0-35-9])\d{8}$/
      if (regMobile.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法的手机号!'))
    }
    return {
      // 用户列表的参数对象
      USerInfo: {
        query: '',
        pagenum: 1,
        pagesize: 5
      },
      // 用户列表
      userList: [],
      // 总人数
      total: 0,
      // 当前所在的页数
      currentPage: 1,
      // 控制修改对话框的显示与隐藏
      editDialogVisible: false,
      // 控制添加用户对话框的显示与隐藏
      addDialogVisible: false,
      // 添加用户的表单数据
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 修改的用户数据
      editForm: {},
      // 添加用户的表单验证规则
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '用户名长度在3到10个字符', trigger: 'blur' }],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '密码长度在6到15个字符', trigger: 'blur' }],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { type: 'email', message: '请输入正确的邮箱地址', trigger: ['blur', 'change'] }],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: cheackMobile, trigger: 'blur' }]
      },
      editFormRules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { type: 'email', message: '请输入正确的邮箱地址', trigger: ['blur', 'change'] }],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: cheackMobile, trigger: 'blur' }]
      }
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    async getUserList() {
      const { data: res } = await this.$http.get('users', {
        params: this.USerInfo
      })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取用户列表失败!')
      }
      this.userList = res.data.users
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      console.log(`每页 ${newSize} 条`)
      this.USerInfo.pagesize = newSize
      this.getUserList()
    },
    handleCurrentChange(newPage) {
      console.log(`当前页: ${newPage}`)
      this.userList.pagenum = newPage
      this.getUserList()
    },
    // 监听用户switch开关状态的改变
    async userStateChanged(userinfo) {
      const { data: res } = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.error('更新用户状态失败!')
      }
      this.$message.success('更新成功!')
    },
    // 监听关闭添加用户窗口时重置表单
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 点击确认按钮，添加新用户
    addUser() {
      this.$refs.addFormRef.validate(async valid => {
        console.log(valid)
        if (!valid) return
        const { data: res } = await this.$http.post('users', this.addForm)
        console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error('创建新用户失败!')
        }
        this.$message.success('用户创建成功!')
        this.addDialogVisible = false
        // 刷新数据列表
        this.getUserList()
      })
    },
    // 修改用户信息
    async showEditDialog(id) {
      this.editDialogVisible = true
      const { data: res } = await this.$http.get('users/' + id)
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户数据失败')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 监听关闭修改窗口时重置表单
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 修改--提交时验证
    editUserInfo() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put('users/' + this.editForm.id, { email: this.editForm.email, mobile: this.editForm.mobile })
        console.log(res)
        if (res.meta.status !== 200) {
          return this.$message.error('修改用户数据失败')
        }
        this.editDialogVisible = false
        // 刷新数据列表
        this.getUserList()
        this.$message.success('更新成功')
      })
    },
    // 根据Id删除对应用户信息
    removeUserById(id) {
      this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete('users/' + id)
        if (res.meta.status !== 200) {
          return this.$message.error('删除用户失败')
        }
        this.$message({
          type: 'success',
          message: '删除成功!'
        })
        // 刷新列表
        this.getUserList()
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    }
  }
}
</script>
