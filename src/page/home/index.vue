<template>
  <n-space vertical>
    <div id="title">App Tracker for Icon Pack Web</div>
    <div style="display: flex; padding-left: 20vw; padding-right: 20vw">
      <n-dropdown
        trigger="hover"
        :options="searchTypeList"
        @select="searchTypeSelect"
        ><n-tag checkable>{{ searchTypeList[searchType].label }}</n-tag>
        <!-- <n-icon size="40">
          <arrow-drop-down-round />
        </n-icon> -->
      </n-dropdown>
      <n-input
        :style="{ 'margin-left': '16px', 'margin-right': '16px' }"
        v-model:value="searchInput"
        :disabled="searchIng"
        :placeholder="searchPlaceHolderList[searchType]"
      />
      <n-button @click="search" :loading="searchIng">搜索</n-button>
    </div>
    <div
      id="search-result"
      style="
        display: flex;
        padding-left: 5vw;
        padding-right: 5vw;
        justify-content: center;
        flex-wrap: wrap;
        background-color: var(--color-background);
      "
    >
      <n-spin v-if="searchIng" size="large" :style="{ 'margin-top': '10vh' }" />
      <app-info-card
        v-if="!searchIng"
        v-for="(item, index) in test"
        :key="index"
        :app-icon="item.appIcon"
        :app-name="item.appName"
        :package-name="item.packageName"
        :activity-name="item.activityName"
        :id="item.id"
        :signature="item.signature"
        :count="item.count"
      />
    </div>
  </n-space>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue'
import {
  NSpace,
  NInput,
  NTag,
  NDropdown,
  NIcon,
  NButton,
  NSpin,
} from 'naive-ui'
import AppInfoCard, { AppInfoCardProps } from '@/components/app-info-card.vue'
import { request } from '@/utils/net'

export default defineComponent({
  components: {
    NSpace,
    NInput,
    NTag,
    NDropdown,
    NIcon,
    AppInfoCard,
    NButton,
    NSpin,
  },
  setup() {
    let searchIng = ref(false)
    let searchInput = ref(null)
    /**
     * 0 正常搜索
     * 1 正则搜索
     * 2 来源搜索
     */
    let searchType = ref(0)
    let searchTypeList = [
      {
        label: '搜索',
        key: 0,
      },
      {
        label: '正则',
        key: 1,
      },
      {
        label: '来源',
        key: 2,
      },
    ]
    let searchPlaceHolderList = Array<string>(
      '请输入关键词',
      '请输入正则',
      '请输入来源'
    )

    // 搜索类型选择
    function searchTypeSelect(key: number) {
      searchType.value = key
      searchInput.value = null
    }

    // 测试数据
    let test = ref(Array<AppInfoCardProps>())

    async function search() {
      searchIng.value = true
      request
        .get({
          url: 'search',
          params: {
            q: searchInput.value,
            page: 1,
            per: 20,
          },
        })
        .then((res) => {
          test.value = res.items
          searchIng.value = false
        })
        .catch((err) => {
          console.log(err)
          searchIng.value = false
        })
    }

    return {
      searchType,
      searchPlaceHolderList,
      searchTypeList,
      searchTypeSelect,
      searchInput,
      search,
      test,
      searchIng,
    }
  },
})
</script>

<style scoped>
#title {
  margin-top: 5vh;
  font-size: 3rem;
  font-weight: bold;
  color: var(--color-text);
}
</style>
