<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert title="注意：只允许为第三级分类设置相关参数！"
                type="warning"
                :closable="false"
                show-icon>
      </el-alert>
      <!-- 选择商品分类区域 -->
      <el-row class="cate_opt">
        <el-col>
          <span>请选择商品分类：</span>
          <!-- 选择商品分类的级联选择器 -->
          <el-cascader v-model="selectedKeys"
                       :options="catelist"
                       :props="cateProps"
                       @change="handleChange"
                       clearable></el-cascader>
        </el-col>
      </el-row>
      <!-- tab 页签区域 -->
      <el-tabs v-model="activeName"
               @tab-click="handleTabClick">
        <!-- 添加动态参数的面板 -->
        <el-tab-pane label="动态参数"
                     name="many">
          <el-button type="primary"
                     size="mini"
                     :disabled="isBtnDisabled"
                     @click="addDialogVisible = true">添加参数</el-button>
          <el-table :data="manyTableData"
                    border
                    stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template v-slot="scope">
                <!-- 循环渲染Tag标签 -->
                <el-tag v-for="(item,i) in scope.row.attr_vals"
                        :key="i"
                        closable
                        @close="handleClose(i,scope.row)">
                  {{item}}
                </el-tag>
                <!-- 显示输入框 -->
                <el-input class="input-new-tag"
                          v-if="scope.row.inputVisible"
                          v-model="scope.row.inputValue"
                          ref="saveTagInput"
                          size="small"
                          @keyup.enter.native="handleInputConfirm(scope.row)"
                          @blur="handleInputConfirm(scope.row)">
                </el-input>
                <!-- 按钮 -->
                <el-button v-else
                           class="button-new-tag"
                           size="small"
                           @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"
                             label="#"></el-table-column>
            <el-table-column label="参数名称"
                             prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template v-slot="scope">
                <el-button type="primary"
                           icon="el-icon-edit"
                           @click="showEidtDialog(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger"
                           icon="el-icon-delete"
                           @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>

        <!-- 添加静态属性的面板 -->
        <el-tab-pane label="静态属性"
                     name="only">
          <el-button type="primary"
                     size="mini"
                     :disabled="isBtnDisabled"
                     @click="addDialogVisible = true">添加属性</el-button>
          <el-table :data="onlyTableData"
                    border
                    stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template v-slot="scope">
                <!-- 循环渲染Tag标签 -->
                <el-tag v-for="(item,i) in scope.row.attr_vals"
                        :key="i"
                        closable
                        @close="handleClose(i,scope.row)">
                  {{item}}
                </el-tag>
                <!-- 显示输入框 -->
                <el-input class="input-new-tag"
                          v-if="scope.row.inputVisible"
                          v-model="scope.row.inputValue"
                          ref="saveTagInput"
                          size="small"
                          @keyup.enter.native="handleInputConfirm(scope.row)"
                          @blur="handleInputConfirm(scope.row)">
                </el-input>
                <!-- 按钮 -->
                <el-button v-else
                           class="button-new-tag"
                           size="small"
                           @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"
                             label="#"></el-table-column>
            <el-table-column label="属性名称"
                             prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template v-slot="scope">
                <el-button type="primary"
                           icon="el-icon-edit"
                           size="mini"
                           @click="showEidtDialog(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger"
                           icon="el-icon-delete"
                           size="mini"
                           @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- 添加参数对话框 -->
    <el-dialog :title="'添加'
                           +titleText"
               :visible.sync="addDialogVisible"
               width="50%"
               @close="addDialogClose">
      <el-form :model="addForm"
               :rules="addFormRules"
               ref="addFormRef"
               label-width="100px">
        <el-form-item :label="titleText"
                      prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="addParams">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改参数对话框 -->
    <el-dialog :title="'修改'+titleText"
               :visible.sync="editDialogVisible"
               width="50%"
               @close="editDialogClose">
      <el-form :model="editForm"
               :rules="addFormRules"
               ref="editFormRef"
               label-width="100px">
        <el-form-item :label="titleText"
                      prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      catelist: [],
      // 级联选择器的配置对象
      cateProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 级联选择器双向绑定的数组
      selectedKeys: [],
      // 被激活页签的名称
      activeName: 'many',
      // 动态参数的表格数据
      manyTableData: [],
      // 静态属性的数据
      onlyTableData: [],
      // 控制 添加参数对话框的显示和隐藏
      addDialogVisible: false,
      // 添加参数 的表单数据
      addForm: {
        attr_name: ''
      },
      // 添加参数的 表单验证规则
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入内容', trigger: 'blur' }
        ]
      },
      // 控制修改参数 对话框的显示与隐藏
      editDialogVisible: false,
      // 修改参数的表单数据
      editForm: {
        attr_name: ''
      }
    }
  },
  created () {
    this.getCateList()
  },
  computed: {
    // 如果按钮需要被禁用，返回true ,否则返回false
    isBtnDisabled () {
      if (this.selectedKeys.length !== 3) {
        return true
      }
      return false
    },
    // 获取所选中的三级分类的Id
    cateId () {
      if (this.selectedKeys.length === 3) {
        return this.selectedKeys[2]
      }
      return null
    },
    // 动态获取 添加参数的标题
    titleText () {
      if (this.activeName === 'many') {
        return '动态参数'
      }
      return '静态属性'
    }
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类列表失败！')
      }
      this.catelist = res.data
      // console.log(this.catelist);
    },
    // 选项发生变化触发这个函数
    handleChange () {
      this.getParamsData()
    },
    // Tab页签的点击事件
    handleTabClick () {
      console.log(this.activeName)
      this.getParamsData()
    },
    // 获取参数列表数据
    async getParamsData () {
      if (this.selectedKeys.length !== 3) {
        this.selectedKeys = []
        this.manyTableData = []
        this.onlyTableData = []
      }
      // 证明选中了三级分类
      // console.log(this.selectedKeys);
      // 根据所选分类的id,和对应所处的面板，获取对应的参数
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        { params: { sel: this.activeName } }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败！')
      }
      res.data.forEach(item => {
        // 文本输入框的显示和隐藏
        item.inputVisible = false
        // 文本输入框的值
        item.inputValue = ''
        // 将数组以 空格 分开
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
      })
      console.log(res.data)

      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    // 监听 添加参数对话框的关闭事件
    addDialogClose () {
      this.$refs.addFormRef.resetFields()
    },
    // 点击确定，添加参数
    addParams () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return

        const { data: res } = await this.$http.post(
          `categories/${this.cateId}/attributes`, {
          attr_name: this.addForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 201) {
          return this.$message.error('添加参数失败！')
        }
        this.$message.success('添加参数成功！')
        this.addDialogVisible = false
        this.getParamsData()
      })
    },
    // 重置修改表单
    editDialogClose () {
      this.$refs.editFormRef.resetFields()
    },
    // 点击确定，修改参数
    editParams () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return

        const { data: res } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
          {
            attr_name: this.editForm.attr_name,
            attr_sel: this.activeName
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('修改参数失败！')
        }
        this.$message.success('修改参数成功！')
        this.getParamsData()
        this.editDialogVisible = false
      })
    },
    // 点击编辑，展示修改参数对话框
    async showEidtDialog (attr_id) {
      // 查询参数信息
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes/${attr_id}`,
        { params: { attr_sel: this.activeName } }
      )

      if (res.meta.status !== 200) {
        return this.$message.error('查询参数信息失败！')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 根据id 删除参数信息
    async removeParams (attr_id) {
      const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      // 取消删除
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      // 执行删除操作
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${attr_id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除参数失败！')
      }
      this.$message.success('删除参数成功！')
      this.getParamsData()
    },
    // 文本框失去焦点，或者按了enter键 触发
    async handleInputConfirm (row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      // 如果没有return,则证明输入的内容，需要做后续处理
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      //  发起请求，保存这次操作
      this.saveAttrVals(row)

    },
    //将对sttr_vals的修改，保存到数据库中
    async saveAttrVals (row) {
      //  发起请求，保存这次操作
      const { data: res } = await this.$http.put(
        `categories/${this.cateId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(' ')
        })
      if (res.meta.status !== 200) {
        return this.$message.error('修改参数项失败！')
      }
      this.$message.success('修改参数项成功！')
    },
    // 显示文本输入框
    showInput (row) {
      row.inputVisible = true
      // 让文本框自动获得焦点
      // $nextTick 方法的作用，就是当页面上元素被重新渲染之后，才会指定回调函数中的代码
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus();
      })
    },
    // 删除tag 参数项
    handleClose (i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    }
  }
}
</script>

<style lang="less" scoped>
.cate_opt {
  margin: 10px 0;
}
.el-tag {
  margin: 10px;
}
.input-new-tag {
  width: 120px;
}
</style>