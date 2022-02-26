<template>
  <div id="info">
    <a style="font-weight: bold; margin-left: 8px; font-size: 1.5rem">
      {{ appName }}
    </a>
    <n-space vertical align="start">
      <div style="display: flex">
        <div
          style="
            display: flex;
            flex-direction: column;
            align-items: flex-start;
            flex-wrap: nowrap;
          "
        >
          <!-- <a style="font-weight: bold; margin-left: 8px; white-space: nowrap"
            >ID:
            <n-tag style="font-weight: normal" checkable @click="copyItem(0)">{{
              id
            }}</n-tag>
          </a>-->
        </div>
      </div>
      <a style="font-weight: bold; margin-left: 8px">
        包名：
        <n-tag style="font-weight: normal" checkable @click="copyItem(2)">
          {{ packageName }}
        </n-tag>
      </a>
      <div style="display: flex">
        <a style="font-weight: bold; margin-left: 8px; white-space: nowrap">
          启动项名：
        </a>
        <n-tag style="font-weight: normal" checkable @click="copyItem(3)">
          {{ activityName }}
        </n-tag>
      </div>

      <a style="font-weight: bold; margin-left: 8px; white-space: nowrap">
        类别：
        <n-tag style="font-weight: normal" checkable @click="copyItem(4)">
          {{ signature }}
        </n-tag>
      </a>
      <a style="font-weight: bold; margin-left: 8px; white-space: nowrap">
        申请次数：
        <n-tag style="font-weight: normal" checkable>{{ count }}</n-tag>
      </a>
    </n-space>
    <n-button :style="{ 'align-self': 'flex-end' }" @click="copyAll"
      >复制</n-button
    >
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import { NImage, NSpace, NButton, useMessage, NTag } from 'naive-ui'
import useClipboard from 'vue-clipboard3'

export interface AppInfoCardProps {
  appIcon: string
  appName: string
  packageName: string
  activityName: string
  id: string
  signature: string
  count: number
}

export default defineComponent({
  components: { NImage, NSpace, NButton, NTag },
  props: {
    appIcon: {
      type: String,
      default: '',
    },
    appName: {
      type: String,
      default: '',
    },
    packageName: {
      type: String,
      default: '',
    },
    activityName: {
      type: String,
      default: '',
    },
    id: {
      type: String,
      default: '',
    },
    signature: {
      type: String,
      default: '',
    },
    count: {
      type: Number,
      default: 0,
    },
  },
  setup(props) {
    const message = useMessage()
    const { toClipboard } = useClipboard()

    // 复制所有信息
    async function copyAll() {
      try {
        await toClipboard(`
{
  "appIcon": "${props.appIcon}",
  "appName": "${props.appName}",
  "packageName": "${props.packageName}",
  "activityName": "${props.activityName}",
  "id": "${props.id}",
  "signature": "${props.signature}"
}
        `)
        message.success('复制成功')
      } catch (e: any) {
        message.error(e.message)
      }
    }

    function copyItem(index: Number) {
      switch (index) {
        case 0:
          toClipboard(props.id)
          break
        case 1:
          toClipboard(props.appName)
          break
        case 2:
          toClipboard(props.packageName)
          break
        case 3:
          toClipboard(props.activityName)
          break
        case 4:
          toClipboard(props.signature)
          break
      }
      message.success('复制成功')
    }

    return {
      copyAll,
      copyItem,
    }
  },
})
</script>

<style>
.n-tag__content {
  white-space: pre-wrap;
  word-break: break-all;
  text-align: justify;
}
</style>

<style scoped>
#info {
  background-color: #fff;
  border-radius: 25px;
  display: flex;
  padding: 16px;
  margin: 16px;
  flex-direction: column;
  width: 300px;
}

@media (max-width: 768px) {
  #info {
    width: auto;
    max-width: 90%;
  }
}
</style>
