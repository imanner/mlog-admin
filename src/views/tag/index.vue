<template>
  <section>
    <!--工具条-->
    <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
      <el-form :inline="true" :model="filters">
        <el-form-item>
          <el-input v-model="filters.name" placeholder="名称" />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="list">查询</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="handleAdd">新增</el-button>
        </el-form-item>
      </el-form>
    </el-col>

    <!--列表-->
    <el-table
      v-loading="listLoading"
      :data="results"
      highlight-current-row
      border
      style="width: 100%;"
    >
      <el-table-column prop="id" label="编号" />
      <el-table-column prop="name" label="名称" />
      <el-table-column prop="description" label="描述" />
      <el-table-column prop="status" label="状态">
        <template slot-scope="scope">{{ scope.row.status === 0 ? '启用' : '禁用' }}</template>
      </el-table-column>
      <el-table-column prop="createTime" label="创建时间">
        <template scope="scope">{{ scope.row.createTime | formatDate }}</template>
      </el-table-column>
      <el-table-column label="操作" width="150">
        <template scope="scope">
          <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!--工具条-->
    <el-col :span="24" class="toolbar">
      <el-pagination
        layout="total, sizes, prev, pager, next, jumper"
        :page-sizes="[20, 50, 100, 300]"
        :current-page="page.page"
        :page-size="page.limit"
        :total="page.total"
        style="float:right;"
      />
    </el-col>

    <!--新增界面-->
    <el-dialog title="新增" :visible.sync="addFormVisible" :close-on-click-modal="false">
      <el-form ref="addForm" :model="addForm" label-width="80px" :rules="addFormRules">
        <el-form-item label="分类" prop="categoryId">
          <el-select v-model="addForm.categoryId" placeholder="请选择">
            <el-option
              v-for="item in options"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="名称" prop="name">
          <el-input v-model="addForm.name" auto-complete="off" />
        </el-form-item>
        <el-form-item label="描述" prop="description">
          <el-input v-model="addForm.description" type="textarea" auto-complete="off" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="addFormVisible = false">取消</el-button>
        <el-button type="primary" :loading="addLoading" @click.native="addSubmit">提交</el-button>
      </div>
    </el-dialog>

    <!--编辑界面-->
    <el-dialog title="编辑" :visible.sync="editFormVisible" :close-on-click-modal="false">
      <el-form ref="editForm" :model="editForm" label-width="80px" :rules="editFormRules">
        <el-input v-model="editForm.id" type="hidden" />
        <el-form-item label="分类" prop="categoryId">
          <el-select v-model="editForm.categoryId" placeholder="请选择">
            <el-option
              v-for="item in options"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="名称" prop="name">
          <el-input v-model="editForm.name" auto-complete="off" />
        </el-form-item>
        <el-form-item label="描述" prop="description">
          <el-input v-model="editForm.description" type="textarea" auto-complete="off" />
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-select v-model="editForm.status" placeholder="请选择">
            <el-option :key="0" :value="0" label="启用" />
            <el-option :key="1" :value="1" label="禁用" />
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="editFormVisible = false">取消</el-button>
        <el-button type="primary" :loading="editLoading" @click.native="editSubmit">提交</el-button>
      </div>
    </el-dialog>
  </section>
</template>

<script>
import TagApi from '../../apis/TagApi'
import CategoryApi from '../../apis/CategoryApi'

export default {
  name: 'List',
  data() {
    return {
      results: [],
      listLoading: false,
      page: {},
      filters: {},

      options: [],

      addForm: {
        categoryId: 0,
        name: '',
        description: ''
      },
      addFormVisible: false,
      addFormRules: {},
      addLoading: false,

      editForm: {
        id: '',
        categoryId: 0,
        name: '',
        description: '',
        status: 0
      },
      editFormVisible: false,
      editFormRules: {},
      editLoading: false
    }
  },
  mounted() {
    this.list()
    this.loadOptions()
  },
  methods: {
    list() {
      const me = this
      me.listLoading = true
      TagApi.list()
        .then(data => {
          me.results = data.results
          me.page = data.page
        })
        .finally(() => {
          me.listLoading = false
        })
    },
    loadOptions() {
      const me = this
      CategoryApi.options()
        .then(data => {
          me.options = data
        })
        .catch(rsp => {
          console.error(rsp)
        })
    },
    handleAdd() {
      this.addForm = {
        categoryId: '',
        name: '',
        description: ''
      }
      this.addFormVisible = true
    },
    addSubmit() {
      const me = this
      TagApi.create(this.addForm)
        .then(data => {
          me.$message({ message: '提交成功', type: 'success' })
          me.addFormVisible = false
          me.list()
        })
        .catch(rsp => {
          me.$notify.error({ title: '错误', message: rsp.message })
        })
    },
    handleEdit(index, row) {
      const me = this
      TagApi.get(row.id)
        .then(data => {
          me.editForm = Object.assign({}, data)
          me.editFormVisible = true
        })
        .catch(rsp => {
          me.$notify.error({ title: '错误', message: rsp.message })
        })
    },
    editSubmit() {
      const me = this
      TagApi.update(me.editForm)
        .then(data => {
          me.list()
          me.editFormVisible = false
        })
        .catch(rsp => {
          me.$notify.error({ title: '错误', message: rsp.message })
        })
    }
  }
}
</script>

<style scoped>

</style>
