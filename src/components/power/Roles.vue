<template>
  <div>
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button
            type="primary"
            class="addRoles"
            @click="addDialogVisible = true"
            >添加角色</el-button
          >
          <el-table :data="rolesList" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-row
                  v-for="(item1, i1) in scope.row.children"
                  :key="item1.id"
                  :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'center']"
                >
                  <!-- 渲染一级 -->
                  <el-col :span="5">
                    <el-tag
                      closable
                      @close="removeRightById(scope.row, item1.id)"
                    >
                      {{ item1.authName }}
                    </el-tag>
                    <i class="el-icon-right"></i>
                  </el-col>
                  <el-col :span="19">
                    <el-row
                      :class="[i2 === 0 ? '' : 'bdtop', 'center']"
                      v-for="(item2, i2) in item1.children"
                      :key="item2.id"
                    >
                      <el-col :span="6">
                        <el-tag
                          type="success"
                          closable
                          @close="removeRightById(scope.row, item2.id)"
                        >
                          {{ item2.authName }}
                        </el-tag>
                        <i class="el-icon-right"></i>
                      </el-col>
                      <el-col :span="18">
                        <el-tag
                          type="warning"
                          v-for="item3 in item2.children"
                          :key="item3.id"
                          closable
                          @close="removeRightById(scope.row, item3.id)"
                          >{{ item3.authName }}</el-tag
                        >
                      </el-col>
                    </el-row>
                  </el-col>
                </el-row>

                <!-- <pre>
                    {{ scope.row }}
                </pre> -->
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="角色名称" prop="roleName"></el-table-column>
            <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <!-- {{scope.row}} -->
                <el-button
                  size="mini"
                  type="primary"
                  icon="el-icon-edit"
                  @click="editShowRoles(scope.row.id)"
                  >编辑</el-button
                >
                <!-- 删除角色 -->
                <el-button
                  size="mini"
                  type="danger"
                  icon="el-icon-delete"
                  @click="deleteRoles(scope.row.id)"
                  >删除</el-button
                >
                <el-button
                  size="mini"
                  type="warning"
                  icon="el-icon-setting"
                  @click="showSetRightDialog(scope.row)"
                  >分配权限</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-col>
      </el-row>
    </el-card>
    <!-- 添加角色对话框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addDialogVisible"
      width="30%"
      @close="addDialogClosed"
    >
      <el-form
        :model="addRolesForm"
        :rules="addRolesRules"
        ref="addRolesRef"
        label-width="80px"
      >
        <!-- 添加内容表格区 -->
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addRolesForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRolesForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRoles">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改角色对话框 -->
    <el-dialog
      title="修改角色"
      :visible.sync="editDialogVisible"
      width="30%"
      @close="editDialogClosed"
    >
      <el-form
        :model="editRolesForm"
        :rules="editRolesRules"
        ref="editRolesRef"
        label-width="80px"
      >
        <!-- 添加内容表格区 -->
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editRolesForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRolesForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoles">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限对话框 -->
    <el-dialog
      title="提示"
      :visible.sync="setRightDialogVisible"
      width="30%"
      @close="setRightDialogClosed"
    >
      <el-tree
        :data="rightslist"
        :props="treeProps"
        show-checkbox
        default-expand-all
        node-key="id"
        :default-checked-keys="defKeys"
        ref="treeRef"
      ></el-tree>

      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      rolesList: [],
      addDialogVisible: false,
      addRolesForm: {
        roleName: '',
        roleDesc: '',
      },
      addRolesRules: {
        roleName: [
          { required: true, message: '请输入角色名', trigger: 'blur' },
          {
            min: 2,
            max: 10,
            message: '角色名的长度在2~10个字符之间',
            trigger: 'blur',
          },
        ],
        roleDesc: [
          {
            min: 2,
            max: 15,
            message: '角色描述的长度在2~15个字符之间',
            trigger: 'blur',
          },
        ],
      },
      editDialogVisible: false,
      editRolesForm: {},
      editRolesRules: {
        roleName: [
          { required: true, message: '请输入角色名', trigger: 'blur' },
          {
            min: 2,
            max: 10,
            message: '角色名的长度在2~10个字符之间',
            trigger: 'blur',
          },
        ],
        roleDesc: [
          {
            min: 2,
            max: 15,
            message: '角色描述的长度在2~15个字符之间',
            trigger: 'blur',
          },
        ],
      },
      setRightDialogVisible: false,
      rightslist: [],
      treeProps: {
        children: 'children',
        label: 'authName',
      },

      defKeys: [],
      //   当前即将分配权限de角色id
      roleId: '',
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      // console.log(s)
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色失败')
      }
      this.rolesList = res.data
      //   this.$message.success('获取角色成功')
    },

    // 监听到关闭对话框重置校验
    addDialogClosed() {
      this.$refs.addRolesRef.resetFields()
    },
    // 点击确定进行校验发请求
    addRoles() {
      this.$refs.addRolesRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.post('roles', this.addRolesForm)
        // console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error('添加角色失败')
        }
        this.addDialogVisible = false
        this.getRolesList()
        this.$message.success('添加角色成功')
      })
    },
    // 编辑角色
    async editShowRoles(id) {
      // console.log(id)
      this.editDialogVisible = true
      const { data: res } = await this.$http.get('roles/' + id)
      console.log(res)

      if (res.meta.status !== 200) {
        return this.$message.error('查询角色失败')
      }
      this.editRolesForm = res.data
    },
    editDialogClosed() {
      this.$refs.editRolesRef.resetFields()
    },
    // 确认编辑
    editRoles() {
      this.$refs.editRolesRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          'roles/' + this.editRolesForm.roleId,
          {
            roleName: this.editRolesForm.roleName,
            roleDesc: this.editRolesForm.roleDesc,
          }
        )
        console.log(res)

        if (res.meta.status !== 200) {
          return this.$message.error('更新角色信息失败')
        }
        this.editDialogVisible = false
        this.getRolesList()
        this.$message.success('更新角色信息成功')
      })
    },

    // 删除角色
    async deleteRoles(id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该角色, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
        }
      ).catch((err) => err)
      console.log(confirmResult)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败')
      }
      this.getRolesList()
      this.$message.success('删除用户成功')
    },

    // 根据id删除相应的权限
    async removeRightById(role, rightId) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
        }
      ).catch((err) => err)
      if (confirmResult !== 'confirm') {
        return this.$message.error('取消了删除')
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      )
      // console.log(res)

      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败')
      }
      // this.getRolesList()
      role.children = res.data

      this.$message.success('删除权限成功')
    },
    // 分配权限
    async showSetRightDialog(role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限数据失败')
      }

      this.rightslist = res.data
      this.getLeafKeys(role, this.defKeys)

      this.setRightDialogVisible = true
    },

    getLeafKeys(node, arr) {
      // 如果当前没有这个节点，就是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }

      node.children.forEach((item) => {
        this.getLeafKeys(item, arr)
      })
    },

    setRightDialogClosed() {
      this.defKeys = []
    },
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys(),
      ]
      // console.log(keys)
      const idStr = keys.join(',')
      // console.log(idStr)

      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败')
      }

      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    },
  },
}
</script>

<style scoped>
.addRoles {
  margin-bottom: 12px;
}
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.center {
  display: flex;
  align-items: center;
}
</style>
