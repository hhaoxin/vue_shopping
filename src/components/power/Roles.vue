<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色--按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表 -->
      <el-table :data="rolesList" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <!-- 栅格系统布局 -->
            <el-row v-for="(item1, index1) in scope.row.children" :key="item1.id" :class="['bodderButtom', index1 === 0 ? 'boderTop' : '', 'vcenter']">
              <!-- 一级权限 -->
              <el-col :span="5">
                <el-tag>{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二、三级权限 -->
              <el-col :span="19">
                <!-- for循环嵌套渲染二级权限 -->
                <el-row v-for="(item2, index2) in item1.children" :key="item2.id" :class="[index2 === 0 ? '' : 'boderTop', 'vcenter']">
                  <!-- 二级 -->
                  <el-col :span="6">
                    <el-tag type="success">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 三级 -->
                  <el-col :span="18">
                    <el-tag type="warning" v-for="(item3, index3) in item2.children" :key="item3.id" :class="[index3 === 0 ? '' : 'boderTop']">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 其他列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" min-width="210px" align="center">
          <template slot-scope="scope">
            <!-- 编辑 -->
            <el-button type="primary" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
            <!-- 删除 -->
            <el-button type="danger" size="mini" @click="removeUserById(scope.row.id)">删除</el-button>
            <!-- 分配权限 -->
            <el-tooltip class="item" effect="dark" content="分配权限" placement="top" :enterable="false">
              <el-button type="info" size="mini">权限</el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 所有角色列表
      rolesList: []
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取失败!')
      }
      this.rolesList = res.data
      this.$message.success('获取成功!')
    }
  }
}
</script>

<style lang="less" scoped>
  .el-tag {
    margin: 6px;
  }
  .boderTop {
    border-top: 1px solid #eee;
  }
  .bodderButtom {
    border-bottom: 1px solid #eee;
  }
  // 展开区内容垂直居中对齐
  .vcenter {
    display: flex;
    align-items: center;
  }
  // 展开区
  .openList {
    // 当窗口缩小时排版不会乱
    width: 100%;
  }
</style>
