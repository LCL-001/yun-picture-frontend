<template>
  <div id="homePage">
    <!-- 搜索框 -->
    <div class="search-bar">
      <a-input-search
        class="home-search-input"
        v-model:value="searchParams.searchText"
        placeholder="从海量图片中搜索"
        enter-button="搜索"
        size="large"
        @search="doSearch"
      />
    </div>
    <!-- 分类和标签筛选 -->
    <a-tabs v-model:active-key="selectedCategory" @change="doSearch">
      <a-tab-pane key="all" tab="全部" />
      <a-tab-pane v-for="category in categoryList" :tab="category" :key="category" />
    </a-tabs>
    <div class="tag-bar">
      <span style="margin-right: 8px">标签：</span>
      <a-space :size="[0, 8]" wrap>
        <a-checkable-tag
          v-for="(tag, index) in tagList"
          :key="tag"
          v-model:checked="selectedTagList[index]"
          @change="doSearch"
        >
          {{ tag }}
        </a-checkable-tag>
      </a-space>
    </div>
    <!-- 图片列表 -->
    <PictureList :dataList="dataList" :loading="loading" />
    <!-- 滚动加载 -->
    <div style="text-align: center; padding: 24px 0">
      <a-spin v-if="loadingMore" size="small" />
      <span v-else-if="!hasMore" style="color: #999; font-size: 13px">— 没有更多了 —</span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onBeforeUnmount, onMounted, reactive, ref } from 'vue'
import {
  listPictureTagCategoryUsingGet,
  listPictureVoByPageUsingPost,
} from '@/api/pictureController.ts'
import { message } from 'ant-design-vue'
import PictureList from '@/components/PictureList.vue' // 定义数据

// 定义数据
const dataList = ref<API.PictureVO[]>([])
const total = ref(0)
const loading = ref(true)

// 搜索条件
const searchParams = reactive<API.PictureQueryRequest>({
  current: 1,
  pageSize: 12,
  sortField: 'createTime',
  sortOrder: 'descend',
})

const loadingMore = ref(false)
const hasMore = ref(true)

// 获取数据
const fetchData = async (append = false) => {
  if (!append) {
    loading.value = true
    searchParams.current = 1
  } else {
    loadingMore.value = true
  }
  const params = {
    ...searchParams,
    tags: [] as string[],
  }
  if (selectedCategory.value !== 'all') {
    params.category = selectedCategory.value
  }
  selectedTagList.value.forEach((useTag, index) => {
    if (useTag) {
      params.tags.push(tagList.value[index])
    }
  })
  const res = await listPictureVoByPageUsingPost(params)
  if (res.data.code === 0 && res.data.data) {
    const records = res.data.data.records ?? []
    if (append) {
      dataList.value.push(...records)
    } else {
      dataList.value = records
    }
    total.value = res.data.data.total ?? 0
    hasMore.value = dataList.value.length < total.value
  } else {
    message.error('获取数据失败，' + res.data.message)
  }
  loading.value = false
  loadingMore.value = false
}

const onScroll = () => {
  const scrollBottom = window.innerHeight + window.scrollY
  const threshold = document.documentElement.scrollHeight - 200
  if (scrollBottom >= threshold && hasMore.value && !loadingMore.value && !loading.value) {
    searchParams.current++
    fetchData(true)
  }
}

onMounted(() => {
  fetchData()
  window.addEventListener('scroll', onScroll, { passive: true })
})

onBeforeUnmount(() => {
  window.removeEventListener('scroll', onScroll)
})

// 搜索
const doSearch = () => {
  searchParams.current = 1
  fetchData()
}

// 标签和分类列表
const categoryList = ref<string[]>([])
const selectedCategory = ref<string>('all')
const tagList = ref<string[]>([])
const selectedTagList = ref<boolean[]>([])

/**
 * 获取标签和分类选项
 * @param values
 */
const getTagCategoryOptions = async () => {
  const res = await listPictureTagCategoryUsingGet()
  if (res.data.code === 0 && res.data.data) {
    tagList.value = res.data.data.tagList ?? []
    categoryList.value = res.data.data.categoryList ?? []
  } else {
    message.error('获取标签分类列表失败，' + res.data.message)
  }
}

onMounted(() => {
  getTagCategoryOptions()
})
</script>

<style scoped>
#homePage {
  margin-bottom: 16px;
}

#homePage .search-bar {
  max-width: 480px;
  margin: 0 auto 16px;
}

#homePage .search-bar :deep(.home-search-input .ant-input-group) {
  display: flex;
  gap: 10px;
}

#homePage .search-bar :deep(.home-search-input .ant-input) {
  height: 42px;
  border-radius: 12px !important;
}

#homePage .search-bar :deep(.home-search-input .ant-input-group-addon) {
  width: auto;
  background: transparent;
}

#homePage .search-bar :deep(.home-search-input .ant-input-group-addon .ant-btn) {
  height: 42px;
  min-width: 88px;
  border-radius: 12px !important;
}

#homePage .tag-bar {
  margin-bottom: 16px;
}
</style>
