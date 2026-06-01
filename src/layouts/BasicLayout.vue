<template>
  <div id="basicLayout">
    <a-layout class="app-frame">
      <a-layout-header class="header">
        <GlobalHeader @toggle-mobile-menu="mobileMenuVisible = !mobileMenuVisible" />
      </a-layout-header>
      <a-layout-content class="content">
        <router-view />
      </a-layout-content>
    </a-layout>

    <!-- 桌面端浮动侧边栏 -->
    <div v-if="loginUserStore.loginUser.id && !isMobile" class="floating-sider" @mouseenter="siderHover = true" @mouseleave="siderHover = false">
      <div class="floating-sider-inner" :class="{ expanded: siderHover }">
        <a-menu
          v-model:selectedKeys="siderCurrent"
          mode="inline"
          :inline-collapsed="!siderHover"
          :items="siderMenuItems"
          @click="doSiderMenuClick"
        />
      </div>
    </div>

    <!-- 回到顶部 -->
    <div v-if="showBackTop" class="back-top-btn" @click="scrollToTop">
      <UpOutlined />
    </div>

    <!-- 移动端底部胶囊导航 -->
    <div v-if="isMobile" class="mobile-bottom-nav">
      <div
        class="bottom-nav-item"
        :class="{ active: currentRoute === '/' || currentRoute === '' }"
        @click="router.push('/')"
      >
        <HomeOutlined />
        <span class="bottom-nav-label">首页</span>
      </div>
      <div
        class="bottom-nav-item"
        :class="{ active: currentRoute === '/square' || currentRoute.startsWith('/square') }"
        @click="router.push('/square')"
      >
        <ReadOutlined />
        <span class="bottom-nav-label">论坛</span>
      </div>
      <div
        class="bottom-nav-item"
        :class="{ active: isMySpaceActive }"
        @click="router.push('/my_space')"
      >
        <FolderOpenOutlined />
        <span class="bottom-nav-label">我的</span>
      </div>
      <!-- 空间tab -->
      <a-popover
        v-if="teamSpaceList.length > 0"
        trigger="click"
        placement="top"
        :open="spacePopoverOpen"
        @openChange="(v: boolean) => spacePopoverOpen = v"
      >
        <div
          class="bottom-nav-item"
          :class="{ active: isTeamSpaceActive }"
          @click="spacePopoverOpen = !spacePopoverOpen"
        >
          <TeamOutlined />
          <span class="bottom-nav-label">空间</span>
        </div>
        <template #content>
          <div class="space-popover-list">
            <div
              v-for="su in teamSpaceList" :key="su.spaceId"
              class="space-popover-item"
              @click="router.push('/space/' + su.spaceId); spacePopoverOpen = false"
            >
              <TeamOutlined style="font-size: 16px; margin-right: 8px" />
              {{ su.space?.spaceName }}
            </div>
            <a-divider style="margin: 4px 0" />
            <div class="space-popover-item" style="color: #1677ff" @click="router.push('/add_space?type=' + SPACE_TYPE_ENUM.TEAM); spacePopoverOpen = false">
              <PlusCircleOutlined style="font-size: 16px; margin-right: 8px" />
              创建团队
            </div>
          </div>
        </template>
      </a-popover>
      <div
        v-else
        class="bottom-nav-item"
        :class="{ active: currentRoute.startsWith('/add_space') }"
        @click="router.push('/add_space?type=' + SPACE_TYPE_ENUM.TEAM)"
      >
        <TeamOutlined />
        <span class="bottom-nav-label">空间</span>
      </div>
    </div>

    <!-- 移动端汉堡抽屉 -->
    <a-drawer
      v-if="loginUserStore.loginUser.id && isMobile"
      v-model:visible="mobileMenuVisible"
      placement="left"
      :width="220"
      :closable="true"
      wrap-class-name="mobile-sider-drawer"
    >
      <a-menu
        v-model:selectedKeys="siderCurrent"
        mode="inline"
        :items="siderMenuItems"
        @click="doSiderMenuClick"
      />
    </a-drawer>
  </div>
</template>

<script setup lang="ts">
import { computed, h, onBeforeUnmount, onMounted, ref, watchEffect } from 'vue'
import { useRouter } from 'vue-router'
import {
  FolderOpenOutlined,
  HomeOutlined,
  PlusCircleOutlined,
  ReadOutlined,
  TeamOutlined,
  UpOutlined,
  UserOutlined,
} from '@ant-design/icons-vue'
import GlobalHeader from '@/components/GlobalHeader.vue'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'
import { useTeamSpaceStore } from '@/stores/useTeamSpaceStore.ts'
import { storeToRefs } from 'pinia'
import { SPACE_TYPE_ENUM } from '@/constants/space.ts'

const router = useRouter()
const loginUserStore = useLoginUserStore()
const teamSpaceStore = useTeamSpaceStore()
const { teamSpaceList } = storeToRefs(teamSpaceStore)

// 响应式
const isMobile = ref(window.innerWidth < 768)
const showBackTop = ref(false)
const onResize = () => { isMobile.value = window.innerWidth < 768 }
const onScroll = () => { showBackTop.value = window.scrollY > 400 }
const scrollToTop = () => { window.scrollTo({ top: 0, behavior: 'smooth' }) }
onMounted(() => {
  window.addEventListener('resize', onResize)
  window.addEventListener('scroll', onScroll)
})
onBeforeUnmount(() => {
  window.removeEventListener('resize', onResize)
  window.removeEventListener('scroll', onScroll)
})

// 侧边栏状态
const siderHover = ref(false)
const mobileMenuVisible = ref(false)
const spacePopoverOpen = ref(false)
const siderCurrent = ref<string[]>(['/'])

// 菜单数据
const fixedMenuItems = [
  { key: '/', icon: () => h(HomeOutlined), label: '首页' },
  { key: '/square', icon: () => h(ReadOutlined), label: '论坛' },
  { key: '/my_space', icon: () => h(UserOutlined), label: '我的空间' },
  { key: '/add_space?type=' + SPACE_TYPE_ENUM.TEAM, icon: () => h(TeamOutlined), label: '创建团队' },
]

const siderMenuItems = computed(() => {
  if (teamSpaceList.value.length < 1) return fixedMenuItems
  const teamSpaceSubMenus = teamSpaceList.value.map((su) => ({
    key: '/space/' + su.spaceId,
    icon: () => h(TeamOutlined),
    label: su.space?.spaceName,
  }))
  return [
    ...fixedMenuItems,
    { type: 'divider' },
    ...teamSpaceSubMenus,
  ]
})

const currentRoute = computed(() => router.currentRoute.value.path)

const isMySpaceActive = computed(() => {
  const p = currentRoute.value
  if (p === '/my_space') return true
  if (p.startsWith('/space/')) {
    const sid = p.replace('/space/', '')
    return !teamSpaceList.value.some(su => String(su.spaceId) === sid)
  }
  return false
})

const isTeamSpaceActive = computed(() => {
  const p = currentRoute.value
  if (p.startsWith('/space/')) {
    const sid = p.replace('/space/', '')
    return teamSpaceList.value.some(su => String(su.spaceId) === sid)
  }
  return p.startsWith('/add_space')
})

const doSiderMenuClick = ({ key }: { key: string }) => {
  router.push(key)
  mobileMenuVisible.value = false
}

// 高亮
watchEffect(() => {
  const p = router.currentRoute.value.path
  if (p === '/') siderCurrent.value = ['/']
  else if (p === '/square') siderCurrent.value = ['/square']
  else if (p === '/my_space') siderCurrent.value = ['/my_space']
  else if (p === '/add_space') siderCurrent.value = ['/my_space']
  else if (p.startsWith('/space/')) {
    const sid = p.replace('/space/', '')
    const found = teamSpaceList.value.some(su => String(su.spaceId) === sid)
    siderCurrent.value = found ? [p] : ['/my_space']
  } else siderCurrent.value = [p]
})

// 加载团队空间
const fetchTeamSpaceList = async () => {
  if (loginUserStore.loginUser.id) {
    await teamSpaceStore.fetchTeamSpaceList()
  }
}

watchEffect(() => { fetchTeamSpaceList() })
</script>

<style scoped>
/* 框架 */
.app-frame {
  min-height: 100vh;
  background:
    radial-gradient(circle at 14% 0%, rgba(186, 224, 255, 0.36), transparent 32%),
    linear-gradient(135deg, #f6faff 0%, #ffffff 52%, #f3f8ff 100%);
}

:deep(.ant-layout) {
  background: transparent;
}

/* 顶部栏 */
.header {
  position: sticky;
  top: 0;
  z-index: 20;
  height: 64px;
  padding-inline: 28px;
  background: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(16px);
  border-bottom: 1px solid rgba(217, 232, 248, 0.8);
  box-shadow: 0 4px 20px rgba(37, 99, 235, 0.04);
  color: unset;
  line-height: 64px;
}

.content {
  min-width: 0;
  padding: 28px 28px 28px 84px;
  background: transparent;
}

/* 浮动侧边栏 */
.floating-sider {
  position: fixed;
  left: 16px;
  top: 50%;
  transform: translateY(-50%);
  z-index: 30;
  max-height: calc(100vh - 160px);
}

.floating-sider-inner {
  width: 52px;
  max-height: calc(100vh - 160px);
  overflow-y: auto;
  overflow-x: hidden;
  background: rgba(255, 255, 255, 0.72);
  backdrop-filter: blur(20px) saturate(140%);
  -webkit-backdrop-filter: blur(20px) saturate(140%);
  border: 1px solid rgba(255, 255, 255, 0.6);
  border-radius: 18px;
  box-shadow:
    0 4px 32px rgba(22, 119, 255, 0.06),
    0 1px 0 rgba(255, 255, 255, 0.8) inset;
  padding: 6px 0;
  transition: width 0.25s cubic-bezier(0.4, 0, 0.2, 1), box-shadow 0.25s ease;
}

.floating-sider-inner.expanded {
  width: 196px;
  box-shadow:
    0 8px 40px rgba(22, 119, 255, 0.1),
    0 2px 0 rgba(255, 255, 255, 0.8) inset;
}

.floating-sider :deep(.ant-menu) {
  background: transparent;
  border: none;
  padding: 0 4px;
}

.floating-sider :deep(.ant-menu-item) {
  height: 38px;
  margin: 1px 4px;
  border-radius: 10px;
  padding: 0 12px !important;
  color: #64748b;
  line-height: 38px;
  font-size: 13px;
  transition: all 0.15s ease;
}

.floating-sider :deep(.ant-menu-item:hover) {
  color: #1677ff;
  background: rgba(22, 119, 255, 0.08);
}

.floating-sider :deep(.ant-menu-item-selected) {
  color: #1677ff;
  background: rgba(22, 119, 255, 0.12);
  font-weight: 600;
}

.floating-sider :deep(.ant-menu-item .anticon) {
  font-size: 20px;
}

.floating-sider :deep(.ant-menu-inline-collapsed) {
  width: 52px;
}

.floating-sider :deep(.ant-menu-inline-collapsed > .ant-menu-item) {
  padding-inline: 0 !important;
  text-align: center;
}

.floating-sider-inner:not(.expanded) :deep(.ant-menu-item-selected) {
  background: transparent;
}

.floating-sider-inner:not(.expanded) :deep(.ant-menu-item-selected .anticon) {
  color: #1677ff;
  background: rgba(22, 119, 255, 0.12);
  border-radius: 50%;
  padding: 6px;
}

.floating-sider :deep(.ant-menu-item-divider) {
  margin: 6px 12px;
  border-color: rgba(0, 0, 0, 0.05);
}

.floating-sider-inner::-webkit-scrollbar {
  width: 0;
}

/* 回到顶部 */
.back-top-btn {
  position: fixed;
  bottom: 24px;
  right: 24px;
  z-index: 40;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.82);
  backdrop-filter: blur(12px);
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  color: #64748b;
  font-size: 16px;
  transition: all 0.2s ease;
}

.back-top-btn:hover {
  color: #1677ff;
  box-shadow: 0 4px 16px rgba(22, 119, 255, 0.15);
  transform: translateY(-2px);
}

/* 底部胶囊导航 */
.mobile-bottom-nav {
  position: fixed;
  bottom: 16px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 30;
  display: flex;
  gap: 4px;
  padding: 6px;
  background: rgba(255, 255, 255, 0.82);
  backdrop-filter: blur(14px);
  border-radius: 24px;
  box-shadow: 0 4px 24px rgba(0, 0, 0, 0.08);
}

.bottom-nav-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 2px;
  width: 56px;
  height: 48px;
  border-radius: 18px;
  cursor: pointer;
  color: #94a3b8;
  transition: color 0.2s, background 0.2s;
  font-size: 20px;
}

.bottom-nav-item.active {
  color: #1677ff;
  background: rgba(22, 119, 255, 0.1);
}

.bottom-nav-label {
  font-size: 10px;
  line-height: 1;
}

/* 空间popover */
.space-popover-list {
  min-width: 160px;
  max-width: 220px;
}

.space-popover-item {
  display: flex;
  align-items: center;
  padding: 8px 4px;
  cursor: pointer;
  border-radius: 6px;
  font-size: 13px;
  color: #334155;
  transition: background 0.15s;
}

.space-popover-item:hover {
  background: rgba(22, 119, 255, 0.06);
}

/* 移动端适配 */
@media (max-width: 768px) {
  .header {
    padding-inline: 14px;
  }

  .content {
    padding: 16px 12px 88px;
  }
}

:deep(.mobile-sider-drawer .ant-drawer-body) {
  padding: 12px 0;
}
</style>
