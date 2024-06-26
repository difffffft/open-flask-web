<template>
  <el-card>
    <el-form :inline="true" :model="state.queryForm" @keyup.enter="getDataList()">
      <el-form-item>
        <el-input v-model="state.queryForm.name" placeholder="附件名称"></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="state.queryForm.platform" placeholder="存储平台"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
      </el-form-item>
      <el-form-item v-auth="'sys:attachment:save'">
        <el-upload :action="BASE_UPLOAD_URL" :headers="headers" :before-upload="beforeUpload"
                   :on-success="handleSuccess" :show-file-list="false">
          <el-button type="primary">上传</el-button>
        </el-upload>
      </el-form-item>
      <el-form-item>
        <el-button v-auth="'sys:attachment:delete'" type="danger" @click="deleteBatchHandle()">删除</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border style="width: 100%"
              @selection-change="selectionChangeHandle">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="name" label="附件名称" show-overflow-tooltip header-align="center"
                       align="center"></el-table-column>
      <el-table-column label="附件地址" show-overflow-tooltip header-align="center" align="center">
        <template #default="scope">
          {{ BASE_URL + scope.row.url }}
        </template>
      </el-table-column>
      <el-table-column prop="size" label="附件大小" header-align="center" align="center">
        <template #default="scope">
          {{ convertSizeFormat(scope.row.size) }}
        </template>
      </el-table-column>
      <el-table-column prop="platform" label="存储平台" header-align="center" align="center"></el-table-column>
      <el-table-column prop="create_time" label="创建时间" header-align="center" align="center"></el-table-column>
      <el-table-column label="操作" fixed="right" header-align="center" align="center" width="150">
        <template #default="scope">
          <el-button type="primary" link @click="downloadHandle(BASE_URL + scope.row.url, scope.row.name)">下载</el-button>
          <el-button v-auth="'sys:attachment:delete'" type="primary" link @click="deleteBatchHandle(scope.row.id)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
        :current-page="state.page"
        :page-sizes="state.pageSizes"
        :page-size="state.limit"
        :total="state.total"
        layout="total, sizes, prev, pager, next, jumper"
        @size-change="sizeChangeHandle"
        @current-change="currentChangeHandle"
    >
    </el-pagination>
  </el-card>
</template>

<script setup name="SysAttachment">
import {ElMessage} from 'element-plus'
import {reactive, ref} from 'vue'

import useCrud from '@/hooks/useCrud'
import useUserStore from "@/store/modules/userStore";
import {convertSizeFormat} from '@/utils/tool'
import {useAttachmentSubmitApi} from '@/api/sys/attachment'
import {BASE_UPLOAD_URL, BASE_URL} from "@/constants";


const {
  state,
  getDataList,
  selectionChangeHandle,
  sizeChangeHandle,
  currentChangeHandle,
  deleteBatchHandle,
  downloadHandle
} = useCrud({
  baseUrl: '/sys/attachment',
  queryForm: {
    name: '',
    platform: ''
  }
})

const userStore = useUserStore()
const headers = ref({
  "Authorization": userStore.token
})

const dataForm = reactive({
  name: '',
  platform: '',
  size: '',
  url: ''
})

const handleSuccess = (res, file) => {
  if (res.code !== 0) {
    ElMessage.error('上传失败：' + res.msg)
    return false
  }
  Object.assign(dataForm, res.data)
  useAttachmentSubmitApi(dataForm).then(() => {
    ElMessage.success({
      message: '上传成功',
      duration: 500,
      onClose: () => {
        getDataList()
      }
    })
  })
}

const beforeUpload = file => {
  if (file.size / 1024 / 1024 / 1024 / 1024 > 1) {
    ElMessage.error('文件大小不能超过100M')
    return false
  }
  return true
}
</script>
