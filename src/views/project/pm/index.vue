<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <el-input v-model="query.blurry" size="small" clearable placeholder="输入项目名称搜索" style="width: 200px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <date-range-picker v-model="query.createTime" class="date-item" />
        <rrOperation />
      </div>
      <crudOperation :permission="permission" />
    </div>
    <!-- 表单渲染 -->
    <el-dialog append-to-body :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="520px">
      <el-form ref="form" :inline="true" :model="form" :rules="rules" size="small" label-width="80px">
        <el-form-item label="项目名称" prop="name">
          <el-input v-model="form.name" style="width: 380px;" />
        </el-form-item>
        <el-form-item label="主持人" prop="host">
          <el-input v-model="form.holder" style="width: 380px;" />
        </el-form-item>
        <el-form-item label="成员" prop="members">
          <el-input v-model="form.members" style="width: 380px;" />
        </el-form-item>
        <el-form-item label="系统运行链接" prop="link">
          <el-input v-model="form.link" style="width: 380px;" />
        </el-form-item>
        <el-form-item label="项目金额" prop="amount">
          <el-input-number v-model.number="form.amount" :min="0" controls-position="right" style="width: 145px;" />
        </el-form-item>
        <el-form-item label="过程性文件" prop="processFiles">
          <el-input v-model="form.hasProcessFile" style="width: 380px;" />
        </el-form-item>
        <el-form-item label="论证材料" prop="materials">
          <el-input v-model="form.hasProofFile" style="width: 380px;" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="text" @click="crud.cancelCU">取消</el-button>
        <el-button :loading="crud.status.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
      </div>
    </el-dialog>
    <el-row :gutter="15">
      <!--项目管理-->
      <el-col :xs="24" :sm="24" :md="24" :lg="24" :xl="24" style="margin-bottom: 10px">
        <el-card class="box-card" shadow="never">
          <div slot="header" class="clearfix">
            <span class="project-span">项目列表</span>
          </div>
          <el-table ref="table" v-loading="crud.loading" highlight-current-row style="width: 100%;" :data="crud.data" @selection-change="crud.selectionChangeHandler" @current-change="handleCurrentChange">
            <el-table-column :selectable="checkboxT" type="selection" width="55" />
            <el-table-column prop="name" label="项目名称" />
            <el-table-column prop="holder" label="主持人" />
            <el-table-column prop="members" label="成员" />
            <el-table-column prop="link" label="系统运行链接" />
            <el-table-column prop="amount" label="项目金额" />
            <el-table-column prop="hasProcessFile" label="过程性文件" />
            <el-table-column prop="hasProofFile" label="论证材料" />
            <el-table-column :show-overflow-tooltip="true" width="135px" prop="createTime" label="创建日期" />
            <el-table-column v-if="checkPer(['admin','projects:edit','projects:del'])" label="操作" width="130px" align="center" fixed="right">
              <template slot-scope="scope">
                <udOperation
                  v-if="scope.row.level >= level"
                  :data="scope.row"
                  :permission="permission"
                />
              </template>
            </el-table-column>
          </el-table>
          <!--分页组件-->
          <pagination />
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import crudProjects from '@/api/system/project'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'
import DateRangePicker from '@/components/DateRangePicker'

const defaultForm = { id: null, name: null, host: null, members: null, link: null, amount: 0, processFiles: null, materials: null }
export default {
  name: 'Project',
  components: { pagination, crudOperation, rrOperation, udOperation, DateRangePicker },
  cruds() {
    return CRUD({ title: '项目', url: 'api/project/query', sort: 'createTime,desc', crudMethod: { ...crudProjects }})
  },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  data() {
    return {
      level: 3,
      currentId: 0,
      permission: {
        add: ['admin', 'projects:add'],
        edit: ['admin', 'projects:edit'],
        del: ['admin', 'projects:del']
      },
      rules: {
        name: [
          { required: true, message: '请输入项目名称', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    // 初始化数据
  },
  methods: {
    // 触发单选
    handleCurrentChange(val) {
      if (val) {
        this.currentId = val.id
      }
    },
    checkboxT(row) {
      return row.level >= this.level
    }
  }
}
</script>

  <style rel="stylesheet/scss" lang="scss">
    .project-span {
      font-weight: bold;color: #303133;
      font-size: 15px;
    }
  </style>
