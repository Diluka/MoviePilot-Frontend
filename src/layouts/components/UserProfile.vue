<script setup lang="ts">
import { useStore } from 'vuex'
import { useConfirm } from 'vuetify-use-dialog'
import { useToast } from 'vue-toast-notification'
import router from '@/router'
import avatar1 from '@images/avatars/avatar-1.png'
import api from '@/api'

// Vuex Store
const store = useStore()

// 确认框
const createConfirm = useConfirm()

// 提示框
const $toast = useToast()

// 进度框
const progressDialog = ref(false)

// 执行注销操作
function logout() {
  // 清除登录状态信息
  store.dispatch('auth/clearToken')

  // 重定向到登录页面或其他适当的页面
  router.push('/login')
}

// 执行重启操作
async function restart() {
  // 弹出提示
  const confirmed = await createConfirm({
    title: '确认',
    content: '确认重启系统吗？',
    confirmationText: '确认',
    cancellationText: '取消',
    dialogProps: {
      maxWidth: '30rem',
    },
  })

  if (confirmed) {
    // 调用API重启
    try {
      // 显示等待框
      progressDialog.value = true
      const result: { [key: string]: any } = await api.get('system/restart')
      if (!result?.success) {
        // 隐藏等待框
        progressDialog.value = false
        // 重启不成功
        $toast.error(result.message)
        return
      }
    }
    catch (error) {
      console.error(error)
    }
    // 注销
    logout()
  }
}

// 从Vuex Store中获取信息
const superUser = store.state.auth.superUser
const userName = store.state.auth.userName
const avatar = store.state.auth.avatar
</script>

<template>
  <VAvatar
    class="cursor-pointer"
    color="primary"
    variant="tonal"
  >
    <VImg :src="avatar ?? avatar1" />

    <!-- SECTION Menu -->
    <VMenu
      activator="parent"
      width="230"
      location="bottom end"
      offset="14px"
    >
      <VList>
        <!-- 👉 User Avatar & Name -->
        <VListItem>
          <template #prepend>
            <VListItemAction start>
              <VAvatar
                color="primary"
                variant="tonal"
              >
                <VImg :src="avatar ?? avatar1" />
              </VAvatar>
            </VListItemAction>
          </template>

          <VListItemTitle class="font-weight-semibold">
            {{ superUser ? "管理员" : "普通用户" }}
          </VListItemTitle>
          <VListItemSubtitle>{{ userName }}</VListItemSubtitle>
        </VListItem>
        <VDivider class="my-2" />

        <!-- 👉 Profile -->
        <VListItem
          v-if="superUser"
          link
          to="setting"
        >
          <template #prepend>
            <VIcon
              class="me-2"
              icon="mdi-account-outline"
              size="22"
            />
          </template>

          <VListItemTitle>设定</VListItemTitle>
        </VListItem>

        <!-- 👉 FAQ -->
        <VListItem
          href="https://github.com/jxxghp/MoviePilot/blob/main/README.md"
          target="_blank"
        >
          <template #prepend>
            <VIcon
              class="me-2"
              icon="mdi-help-circle-outline"
              size="22"
            />
          </template>

          <VListItemTitle>帮助</VListItemTitle>
        </VListItem>

        <!-- Divider -->
        <VDivider class="my-2" />

        <!-- 👉 restart -->
        <VListItem @click="restart">
          <template #prepend>
            <VIcon
              class="me-2"
              icon="mdi-restart"
              size="22"
            />
          </template>

          <VListItemTitle>重启</VListItemTitle>
        </VListItem>

        <!-- 👉 Logout -->
        <VListItem @click="logout">
          <template #prepend>
            <VIcon
              class="me-2"
              icon="mdi-logout"
              size="22"
            />
          </template>

          <VListItemTitle>注销</VListItemTitle>
        </VListItem>
      </VList>
    </VMenu>
    <!-- !SECTION -->
  </VAvatar>
  <!-- 重启进度框 -->
  <vDialog
    v-model="progressDialog"
    width="25rem"
  >
    <vCard
      color="primary"
    >
      <vCardText class="text-center">
        正在重启 ...
        <vProgressLinear
          indeterminate
          color="white"
          class="mb-0 mt-1"
        />
      </vCardText>
    </vCard>
  </vDialog>
</template>
