<template>
  <div class="p-10px">
    <el-form v-show="showSearch" :model="queryParams" :inline="true" label-width="68px">
      <el-form-item label="登录地址" prop="ipaddr">
        <el-input
          v-model.trim="queryParams.ipaddr"
          placeholder="请输入登录地址"
          clearable
          class="wi-150px"
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="用户名称" prop="userName">
        <el-select v-model="queryParams.userName" placeholder="请选择用户名称" clearable class="wi-150px">
          <el-option v-for="dict in sys_normal_disable" :key="dict.dictValue" :label="dict.dictLabel" :value="dict.dictValue" />
        </el-select>
      </el-form-item>
      <el-form-item label="状态" prop="status">
        <el-select v-model="queryParams.status" placeholder="请选择状态" clearable class="wi-150px">
          <el-option v-for="dict in sys_common_status" :key="dict.dictValue" :label="dict.dictLabel" :value="dict.dictValue" />
        </el-select>
      </el-form-item>
      <el-form-item label="登录时间" style="width: 150px}">
        <el-date-picker
          v-model="dateRange"
          value-format="YYYY-MM-DD HH:mm:ss"
          type="daterange"
          range-separator="-"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          :default-time="[new Date(2000, 1, 1, 0, 0, 0), new Date(2000, 1, 1, 23, 59, 59)]"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="Search" @click="handleQuery">搜索</el-button>
        <el-button icon="Refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>
    <el-row :gutter="10" class="mb8">
      <el-button type="danger" plain icon="Delete" :disabled="multiple" @click="handleDelete">删除</el-button>

      <el-button type="danger" plain icon="Delete" @click="handleClean">清空</el-button>
      <el-button type="primary" plain icon="Unlock" :disabled="single" @click="handleUnlock">解锁</el-button>
      <el-button type="warning" plain icon="Download" @click="handleExport">导出</el-button>

      <RightToolBar v-model:showSearch="showSearch" @queryTable="getList" />
      <ColumnFilter v-if="loginInfoList.length" :is-operation="true" :cols="tableHeadColumns" @colChange="colChange" />
    </el-row>
    <el-table
      ref="refElTable"
      v-loading="loading"
      height="calc(100vh - 368px)"
      border
      :data="loginInfoList"
      @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" width="50" align="center" />
      <!--column头字段-->
      <template v-for="item in tableHeadColumns">
        <el-table-column
          v-if="!item.isTemplate && item.showColumn"
          :key="item.prop"
          show-overflow-tooltip
          v-bind="item"
          :align="item.align || 'center'"
          :prop="item.prop"
          :label="item.label"
        />
        <!--登录状态-->
        <el-table-column
          v-if="item.prop === 'status' && item.isTemplate && item.showColumn"
          :key="item.prop"
          show-overflow-tooltip
          v-bind="item"
          align="center"
          :prop="item.prop"
          :label="item.label"
        >
          <template #default="{ row }">
            <dict-tag :options="sys_common_status" :value="row.status" />
          </template>
        </el-table-column>
      </template>

      <el-table-column label="操作" align="center" width="150" class-name="small-padding fixed-width">
        <template #default="{ row }">
          <el-tooltip content="删除" placement="top">
            <el-button link type="primary" icon="Delete" size="large" @click="handleDelete(row)" />
          </el-tooltip>
        </template>
      </el-table-column>
    </el-table>
    <div class="columnSE">
      <Pagination
        v-show="totalNum > 10"
        v-model:page="queryParams.pageNum"
        v-model:limit="queryParams.pageSize"
        :total="totalNum"
        @pagination="getList"
      />
    </div>
  </div>
</template>
<script setup>
import { cleanLoginInfo, listReq, unlockLoginInfo } from '@/api/loginInfo'
import { useDict } from '@/hooks/use-data-dict'
import { onMounted, reactive, ref } from 'vue'
//导入当前页面封装方法
import RightToolBar from '@/components/RightToolBar.vue'
import ColumnFilter from '@/components/ColumnFilter.vue'
/*查询模块*/
const queryParams = reactive({
  pageNum: 1,
  pageSize: 10,
  ipaddr: '', //登录地址
  userName: '', //用户名称
  status: '', //状态
  beginTime: '', //登录时间开始时间
  endTime: '' //登录时间结束时间
})
//备份数据
const bakQueryParams = JSON.stringify(queryParams)
const dateRange = ref([])

//解锁
const handleUnlock = () => {
  unlockLoginInfo()
}

//清空
const handleClean = () => {
  elConfirm('确定', '是否确认清空所有登录日志数据项')
    .then(() => {
      return cleanLoginInfo()
    })
    .then(() => {
      handleQuery()
      elMessage('清空成功')
    })
    .catch(() => {})
}
//查询
const handleQuery = () => {
  queryParams.pageNum = 1
  getList(queryParams)
}
//重置
const resetQuery = () => {
  resetData(queryParams, bakQueryParams)
  dateRange.value = []
  handleQuery()
}
const getList = () => {
  loading.value = true
  if (dateRange.value?.length) {
    queryParams.beginTime = dateRange.value.at(-2)
    queryParams.endTime = dateRange.value.at(-1)
  } else {
    queryParams.beginTime = ''
    queryParams.endTime = ''
  }
  listReq(removeEmptyKey(queryParams)).then(({ rows, total }) => {
    loading.value = false
    loginInfoList.value = rows
    totalNum.value = total
  })
}
onMounted(() => {
  handleQuery()
})
//字典数据
// eslint-disable-next-line camelcase
const { sys_normal_disable, sys_common_status } = useDict(['sys_normal_disable', 'sys_common_status'])

///导入当前页面封装方法
import { colChange, currentHook, handleAdd, handleSelectionChange, removeEmptyKey } from './index-hook'
import { resetData } from '@/hooks/use-common'
import { elConfirm, elMessage } from '@/hooks/use-element.ts'

const {
  refAddEditModal,
  refElTable,
  refExport,
  single,
  multiple,
  ids,
  totalNum,
  loading,
  loginInfoList,
  showSearch,
  tableHeadColumns,
  handleExport,
  handleDelete
} = currentHook(queryParams, getList)
</script>
