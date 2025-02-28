<script setup lang="ts">
import { ref } from 'vue'
import { useToast } from 'vue-toast-notification'
import { useConfirm } from 'vuetify-use-dialog'
import { numberValidator, requiredValidator } from '@/@validators'
import api from '@/api'
import type { TransferHistory } from '@/api/types'
import TmdbSelectorCard from '@/components/cards/TmdbSelectorCard.vue'

// 确认框
const createConfirm = useConfirm()

// 提示框
const $toast = useToast()

// 重新整理对话框
const redoDialog = ref(false)

// TMDB编号
const redoTmdbId = ref('')

// 类型
const redoType = ref('电影')

// 类型下拉框：电影、电视剧
const redoTypeItems = ref([
  { title: '电影', value: '电影' },
  { title: '电视剧', value: '电视剧' },
])

// 当前操作记录
const currentHistory = ref<TransferHistory>()

// 已选中的数据
const selected = ref<TransferHistory[]>([])

// 表头
const headers = [
  { title: '标题', key: 'title', sortable: false },
  { title: '目录', key: 'src', sortable: false },
  { title: '转移方式', key: 'mode', sortable: false },
  { title: '时间', key: 'date', sortable: false },
  { title: '状态', key: 'status', sortable: false },
  { title: '失败原因', key: 'errmsg', sortable: false },
  { title: '', key: 'actions', sortable: false },
]

// 数据列表
const dataList = ref<TransferHistory[]>([])

// 搜索
const search = ref('')

// 加载状态
const loading = ref(false)

// 总条数
const totalItems = ref(0)

// 每页条数
const itemsPerPage = ref(25)

// 当前页码
const currentPage = ref(1)

// 进度条
const progressDialog = ref(false)

// 进度文本
const progressText = ref('请稍候 ...')

// 进度值
const progressValue = ref(0)

// TMDB选择对话框
const tmdbSelectorDialog = ref(false)

// 获取订阅列表数据
async function fetchData({
  page,
  itemsPerPage,
}: {
  page: number
  itemsPerPage: number
}) {
  loading.value = true
  try {
    currentPage.value = page

    const result: { [key: string]: any } = await api.get('history/transfer', {
      params: {
        page,
        count: itemsPerPage,
        title: search.value,
      },
    })

    dataList.value = result.data.list
    totalItems.value = result.data.total
  }
  catch (error) {
    console.error(error)
  }
  loading.value = false
}

// 根据 type 返回不同的图标
function getIcon(type: string) {
  if (type === '电影')
    return 'mdi-movie'
  else if (type === '电视剧')
    return 'mdi-television-classic'
  else
    return 'mdi-help-circle'
}

// 计算颜色
function getStatusColor(status: boolean) {
  return status ? 'success' : 'error'
}

// 转移方式字典
const TransferDict: { [key: string]: string } = {
  copy: '复制',
  move: '移动',
  link: '硬链接',
  softlink: '软链接',
}

// 删除历史记录
async function removeHistory(item: TransferHistory) {
  const isConfirmed = await createConfirm({
    title: '确认',
    content: `同步删除 ${item.title} 对应的媒体库文件 ?`,
    confirmationText: '同步删除文件',
    cancellationText: '仅删除历史记录',
    dialogProps: {
      maxWidth: '50rem',
    },
    confirmationButtonProps: {
      color: 'error',
    },
  })
  if (isConfirmed === undefined)
    return

  // 执行删除
  remove(item, isConfirmed || false)
  // 清空选中项
  selected.value = []
}

// 调用API删除记录
async function remove(item: TransferHistory, deleteFile: boolean) {
  try {
    // 调用删除API
    const result: { [key: string]: any } = await api.delete(`history/transfer?delete_file=${deleteFile}`, {
      data: item,
    })

    if (result.success) {
      fetchData({
        page: currentPage.value,
        itemsPerPage: itemsPerPage.value,
      })
    }
    else {
      $toast.error(`删除失败: ${result.msg}`)
    }
  }
  catch (error) {
    console.error(error)
  }
}

// 批量删除历史记录
async function removeHistoryBatch() {
  if (selected.value.length === 0)
    return
  // 确认
  const isConfirmed = await createConfirm({
    title: '确认',
    content: `同步删除 ${selected.value.length} 条记录对应的媒体库文件 ?`,
    confirmationText: '同步删除文件',
    cancellationText: '仅删除历史记录',
    dialogProps: {
      maxWidth: '50rem',
    },
    confirmationButtonProps: {
      color: 'error',
    },
  })
  if (isConfirmed === undefined)
    return

  console.log(selected.value)

  // 总条数
  const total = selected.value.length
  // 已处理条数
  let handled = 0
  // 显示进度条
  progressDialog.value = true
  // 循环调用removeHistory
  for (const item of selected.value) {
    // 开始删除
    progressText.value = `正在删除 ${item.title} ${item.seasons}${item.episodes} ...`
    await remove(item, isConfirmed || false)
    // 删除完成
    handled++
    progressValue.value = handled / total * 100
  }
  // 清空选中项
  selected.value = []
  // 隐藏进度条
  progressDialog.value = false
  // 重新获取数据
  fetchData({
    page: currentPage.value,
    itemsPerPage: itemsPerPage.value,
  })
}

// 重新整理
async function rehandleHistory() {
  try {
    if (!redoTmdbId.value || !redoType.value)
      return

    redoDialog.value = false
    $toast.info(`正在重新整理 ${currentHistory.value?.title} ...`)

    // 调用API接口重新转移
    const requestData = {
      ...currentHistory.value,
    }

    const result: { [key: string]: any } = await api.post(
      'history/transfer',
      requestData,
      {
        params: {
          mtype: redoType.value,
          new_tmdbid: parseInt(redoTmdbId.value),
        },
      },
    )

    if (result.success) {
      fetchData({
        page: currentPage.value,
        itemsPerPage: itemsPerPage.value,
      })
    }
    else {
      $toast.error(`重新整理失败: ${result.message}！`)
    }
  }
  catch (e) {
    console.log(e)
  }
}

// 弹出菜单
const dropdownItems = ref([
  {
    title: '重新整理',
    value: 1,
    props: {
      prependIcon: 'mdi-redo-variant',
      click: (item: TransferHistory) => {
        redoDialog.value = true
        currentHistory.value = item
      },
    },
  },
  {
    title: '删除',
    value: 2,
    props: {
      prependIcon: 'mdi-trash-can-outline',
      color: 'error',
      click: removeHistory,
    },
  },
])
</script>

<template>
  <VCard class="pb-5">
    <VCardItem>
      <VCardTitle>
        <VRow>
          <VCol> 历史记录 </VCol>
          <VCol>
            <VTextField
              key="search_navbar"
              v-model="search"
              class="text-disabled"
              density="compact"
              label="搜索"
              append-inner-icon="mdi-magnify"
              variant="solo-filled"
              single-line
              hide-details
              flat
              rounded
            />
          </VCol>
        </VRow>
      </VCardTitle>
    </VCardItem>
    <VDataTableServer
      v-model="selected"
      v-model:items-per-page="itemsPerPage"
      :headers="headers"
      :items="dataList"
      :items-length="totalItems"
      :search="search"
      :loading="loading"
      density="compact"
      item-value="id"
      return-object
      fixed-header
      show-select
      items-per-page-text="每页条数"
      page-text="{0}-{1} 共 {2} 条"
      @update:options="fetchData"
    >
      <template #item.title="{ item }">
        <div class="d-flex">
          <VAvatar><VIcon :icon="getIcon(item.raw.type || '')" /></VAvatar>
          <div class="d-flex flex-column ms-1">
            <span class="d-block whitespace-nowrap text-high-emphasis">
              {{ item.raw.title }} {{ item.raw.seasons }}{{ item.raw.episodes }}
            </span>
            <small>{{ item.raw.category }}</small>
          </div>
        </div>
      </template>
      <template #item.src="{ item }">
        <small>{{ item.raw.src }} <br>=> {{ item.raw.dest }}</small>
      </template>
      <template #item.mode="{ item }">
        <VChip
          variant="outlined"
          color="primary"
          size="small"
        >
          {{
            TransferDict[item.raw.mode]
          }}
        </VChip>
      </template>
      <template #item.status="{ item }">
        <VChip
          :color="getStatusColor(item.raw.status)"
          size="small"
        >
          {{ item.raw.status ? "成功" : "失败" }}
        </VChip>
      </template>
      <template #item.date="{ item }">
        <small>{{ item.raw.date }}</small>
      </template>
      <template #item.errmsg="{ item }">
        <small class="text-error">{{ item.raw.errmsg }}</small>
      </template>
      <template #item.actions="{ item }">
        <IconBtn>
          <VIcon icon="mdi-dots-vertical" />
          <VMenu
            activator="parent"
            close-on-content-click
          >
            <VList>
              <VListItem
                v-for="(menu, i) in dropdownItems"
                :key="i"
                variant="plain"
                :base-color="menu.props.color"
                @click="menu.props.click(item.raw)"
              >
                <template #prepend>
                  <VIcon :icon="menu.props.prependIcon" />
                </template>
                <VListItemTitle v-text="menu.title" />
              </VListItem>
            </VList>
          </VMenu>
        </IconBtn>
      </template>
      <template #no-data>
        没有数据
      </template>
    </VDataTableServer>
  </VCard>
  <VDialog
    v-model="redoDialog"
    max-width="50rem"
  >
    <!-- Dialog Content -->
    <VCard title="重新整理">
      <VCardText>
        <VRow>
          <VCol cols="12" md="4">
            <VSelect
              v-model="redoType"
              label="类型"
              :rules="[requiredValidator]"
              :items="redoTypeItems"
            />
          </VCol>
          <VCol cols="12" md="8">
            <VTextField
              v-model="redoTmdbId"
              label="TMDB编号"
              :rules="[requiredValidator, numberValidator]"
              append-inner-icon="mdi-magnify"
              @click:append-inner="tmdbSelectorDialog = true"
            />
          </VCol>
        </VRow>
      </VCardText>

      <VCardActions>
        <VSpacer />
        <VBtn
          @click="rehandleHistory"
          @keydown.enter="rehandleHistory"
        >
          确定
        </VBtn>
      </VCardActions>
    </VCard>
  </VDialog>
  <span v-if="selected.length > 0" class="fixed right-5 bottom-5">
    <VBtn
      icon="mdi-trash-can-outline"
      color="error"
      size="x-large"
      @click="removeHistoryBatch"
    />
  </span>
  <!-- 进度框 -->
  <vDialog
    v-model="progressDialog"
    :scrim="false"
    width="25rem"
  >
    <vCard
      color="primary"
    >
      <vCardText class="text-center">
        {{ progressText }}
        <vProgressLinear
          color="white"
          class="mb-0 mt-1"
          :model-value="progressValue"
        />
      </vCardText>
    </vCard>
  </vDialog>
  <!-- TMDB ID搜索框 -->
  <vDialog
    v-model="tmdbSelectorDialog"
    width="600"
    scrollable
  >
    <TmdbSelectorCard
      v-model="redoTmdbId"
      @close="tmdbSelectorDialog = false"
    />
  </vDialog>
</template>

<style lang="scss">
.v-table th {
  white-space: nowrap;
}
</style>
