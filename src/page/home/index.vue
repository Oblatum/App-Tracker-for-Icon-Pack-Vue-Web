<template>
  <n-space vertical align="center">
    <div id="title">App Tracker for Icon Pack Web</div>
    <div id="searchbar">
      <n-checkbox
        v-model:checked="useRegex"
        :disabled="disableUseRegex"
        @update:checked="updateUseRegex"
      >
        使用正则
      </n-checkbox>
      <n-input
        :style="{ 'margin-left': '16px', 'margin-right': '16px' }"
        v-model:value="searchInput"
        :disabled="searchIng"
        :placeholder="searchPlaceHolderList[searchPlaceHolder]"
      >
        <template #prefix>
          <n-dropdown :options="searchTypeList" @select="searchTypeSelect"
            ><n-tag checkable>{{
              searchTypeList[searchTypeListSelect].label
            }}</n-tag>
            <!-- <n-icon size="40">
          <arrow-drop-down-round />
        </n-icon> -->
          </n-dropdown>
        </template>
      </n-input>
      <n-button @click="searchBtnClick" :loading="searchIng">搜索</n-button>
    </div>
    <n-pagination v-if="showNav" v-model:page="currPage" :page-count="count" />
    <div id="search-result">
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
    <n-pagination
      v-if="showNav"
      v-model:page="currPage"
      :page-count="count"
      style="margin-bottom: 10px"
    />
  </n-space>
</template>

<script lang="ts">
import { defineComponent, ref, watch } from 'vue'
import {
  NSpace,
  NInput,
  NTag,
  NDropdown,
  NIcon,
  NButton,
  NSpin,
  NPagination,
  NCheckbox,
  useMessage,
} from 'naive-ui'
import AppInfoCard, { AppInfoCardProps } from '@/components/app-info-card.vue'
import { request } from '@/utils/net'
import ceil from 'mathjs'

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
    NPagination,
    NCheckbox,
  },
  setup() {
    let message = useMessage()

    let count = ref(0)
    let currPage = ref(1)
    let currSearch = ref(null)

    let showNav = ref(false)

    let disableUseRegex = ref(false)
    let useRegex = ref(false)
    let searchIng = ref(false)
    let searchInput = ref(null)
    /**
     * 0 正常搜索
     * 1 正则搜索
     * 2 来源搜索
     */
    let searchType = ref(0)

    let searchPlaceHolder = ref(0)
    let searchTypeListSelect = ref(0)
    let searchTypeList = [
      {
        label: '搜索',
        key: 0,
      },
      {
        label: '来源',
        key: 1,
      },
    ]
    let searchPlaceHolderList = Array<string>(
      '请输入关键词',
      '请输入正则',
      '请输入来源'
    )

    // 搜索类型选择
    function searchTypeSelect(key: number) {
      if (key == 0) {
        if (searchType.value == 0) {
          disableUseRegex.value = false
          searchType.value = 0
          searchPlaceHolder.value = 0
        } else {
          disableUseRegex.value = false
          searchType.value = 1
          searchPlaceHolder.value = 1
        }
      } else {
        disableUseRegex.value = true
        searchType.value = 2
        searchPlaceHolder.value = 2
      }
      console.log(disableUseRegex.value)
      searchInput.value = null
    }

    function updateUseRegex(value: boolean) {
      if (value) {
        searchType.value = 1
        searchPlaceHolder.value = 1
      } else {
        searchType.value = 0
        searchPlaceHolder.value = 0
      }
    }

    // 测试数据
    let test = ref(Array<AppInfoCardProps>())

    function searchBtnClick() {
      currSearch.value = searchInput.value
      currPage.value = 1
      search(0, searchInput.value)
    }

    function search(page: number, input: string | null) {
      if (searchInput.value == null) {
        message.error('请输入关键词')
        return
      }
      if (useRegex.value) {
        searchRegex(page, input)
      } else {
        searchNormal(page, input)
      }
    }

    async function searchNormal(page: number, input: string | null) {
      searchIng.value = true
      request
        .get({
          url: 'search',
          params: {
            q: input,
            page: page,
            per: 20,
          },
        })
        .then((res) => {
          test.value = res.items
          count.value = Math.ceil(res.metadata.total / 20)
          showNav.value = true
          searchIng.value = false
        })
        .catch((err) => {
          console.log(err)
          searchIng.value = false
        })
    }

    async function searchRegex(page: number, input: string | null) {
      searchIng.value = true
      request
        .get({
          url: 'search/regex',
          params: {
            q: input,
            page: page,
            per: 20,
          },
        })
        .then((res) => {
          test.value = res.items
          count.value = Math.ceil(res.metadata.total / 20)
          showNav.value = true
          searchIng.value = false
        })
        .catch((err) => {
          console.log(err)
          searchIng.value = false
        })
    }

    watch(currPage, (newVal, oldVal) => {
      if (newVal != oldVal) {
        if (currSearch.value != null) {
          search(newVal, currSearch.value)
        }
      }
    })

    return {
      searchPlaceHolderList,
      searchTypeList,
      searchTypeSelect,
      searchTypeListSelect,
      disableUseRegex,
      searchInput,
      search,
      test,
      searchIng,
      useRegex,
      updateUseRegex,
      searchPlaceHolder,
      count,
      currPage,
      showNav,
      searchBtnClick,
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
#searchbar {
  display: flex;
  padding-left: 20vw;
  padding-right: 20vw;
}

#search-result {
  display: flex;
  padding-left: 5vw;
  padding-right: 5vw;
  justify-content: center;
  flex-wrap: wrap;
  background-color: var(--color-background);
}

@media (max-width: 768px) {
  #searchbar {
    padding-left: 5vw;
    padding-right: 5vw;
  }

  #title {
    margin-top: 5vh;
    font-size: 1.5rem;
    font-weight: bold;
    color: var(--color-text);
  }
}
</style>
