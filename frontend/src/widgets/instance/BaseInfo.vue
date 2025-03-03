<script setup lang="ts">
import { ref } from "vue";
import type { LayoutCard } from "@/types";
import { useLayoutCardTools } from "../../hooks/useCardTools";
import { onMounted, computed } from "vue";
import { t } from "@/lang/i18n";
import { useInstanceInfo } from "@/hooks/useInstance";
import { CheckCircleOutlined, ExclamationCircleOutlined } from "@ant-design/icons-vue";
import { GLOBAL_INSTANCE_NAME } from "../../config/const";
import { parseTimestamp } from "../../tools/time";
import { dockerPortsArray } from "@/tools/common";
import DockerInfo from "./dialogs/DockerInfo.vue";
import { Icon } from 'tdesign-icons-vue-next';

const props = defineProps<{
  card: LayoutCard;
}>();

const DockerInfoDialog = ref<InstanceType<typeof DockerInfo>>();
const { getMetaOrRouteValue } = useLayoutCardTools(props.card);

const instanceId = getMetaOrRouteValue("instanceId");
const daemonId = getMetaOrRouteValue("daemonId");

const { statusText, isRunning, isStopped, instanceTypeText, instanceInfo, execute } =
  useInstanceInfo({
    instanceId,
    daemonId,
    autoRefresh: true
  });

const getInstanceName = computed(() => {
  if (instanceInfo.value?.config.nickname === GLOBAL_INSTANCE_NAME) {
    return t("TXT_CODE_5bdaf23d");
  } else {
    return instanceInfo.value?.config.nickname;
  }
});

const instanceGameServerInfo = computed(() => {
  if (instanceInfo.value?.info?.mcPingOnline) {
    return {
      players: `${instanceInfo.value?.info.currentPlayers} / ${instanceInfo.value?.info.maxPlayers}`,
      version: instanceInfo.value?.info.version
    };
  } else {
    return null;
  }
});

onMounted(async () => {
  if (instanceId && daemonId) {
    await execute({
      params: {
        uuid: instanceId,
        daemonId: daemonId
      }
    });
  }
});
</script>

<template >
  <!-- eslint-disable vue/html-indent -->
<div class="cont">
  <CardPanel class="containerWrapper" style="height: 100%">
    <template #title>
     实例信息
    </template>
    <template #body>
      <a-typography-paragraph>
        <a-typography-text :title="instanceInfo?.instanceUuid">
          <div class="info-a"> {{ t("TXT_CODE_30051f9b") }}</div>
        </a-typography-text>
        <div class="info-b"><a-typography-text :copyable="{ text: instanceInfo?.instanceUuid }"> </a-typography-text>
        </div>
      </a-typography-paragraph>

      <a-typography-paragraph>
        <div class="info-a">实例名称：</div><div class="info-b">{{ getInstanceName }}</div>
      </a-typography-paragraph>
        
   
         <a-typography-paragraph v-if="isRunning" color="green" >
          <div class="info-a">实例状态:</div>
          <div class="info-b">
            <icon class="statuscircle" size="18px" name="round" style="color: green" />
            {{ statusText }}</div>
        </a-typography-paragraph><a-typography-paragraph v-else-if="isStopped" >
          <div class="info-a">实例状态:</div><div class="info-b"> <icon class="statuscircle" size="18px" name="round" style="color: red" /> {{ statusText }}
          </div>
          </a-typography-paragraph>
          <a-typography-paragraph v-else >
            <div class="info-a">实例状态: </div><div class="info-b">{{ statusText }}</div>
          </a-typography-paragraph>
     
      <a-typography-paragraph>
       <a-typography-text class="" :title="daemonId">
        <div class="info-a">{{ t("TXT_CODE_5f2d2e30") }}</div>
        </a-typography-text>
        <div class="info-b"> <a-typography-text :copyable="{ text: daemonId }"> </a-typography-text></div>
      </a-typography-paragraph> 
      <a-typography-paragraph>
      <div class="info-a"> 实例类型：</div>    <div class="info-b"> {{ instanceTypeText }}</div>
      </a-typography-paragraph>
     
      <a-typography-paragraph>
        <div class="info-a">{{ t("TXT_CODE_ae747cc0") }}</div>
        <div class="info-b">{{ parseTimestamp(instanceInfo?.config.endTime) || t("TXT_CODE_e3a77a77") }}</div>
      </a-typography-paragraph>

      <a-typography-paragraph
        v-if="
          instanceInfo?.config.processType === 'docker' &&
          Number(instanceInfo?.config.docker.ports?.length) > 0
        "
      >
      <div class="info-a">{{ t("TXT_CODE_2e4469f6") }}</div>
        <div style="padding: 10px 0px 0px 16px">
          <div
            v-for="(item, index) in dockerPortsArray(instanceInfo?.config.docker.ports ?? [])"
            :key="index"
            class="mb-4"
          >
            <span>
              <a-tag color="green">{{ item.protocol.toUpperCase() }}</a-tag>
            </span>
            <a-tag>
              <span>{{ t("TXT_CODE_8dfc41ef") }}: {{ item.host }}</span>
              <span class="ml-4"> {{ t("TXT_CODE_8f8103b7") }}: {{ item.container }} </span>
            </a-tag>
          </div>
        </div>
      </a-typography-paragraph>
    </template>
  </CardPanel>

  <CardPanel class="containerWrapper" style="height: 100%">
  <template #title>
    游戏信息
  </template>
  <template #body>
    <a-typography-paragraph>
      <div class="info-a">启动次数：</div>
      <div class="info-b">{{ instanceInfo?.started }}</div>
    </a-typography-paragraph>

    <a-typography-paragraph>
      <div class="info-a">实例标签：</div>
      <div class="info-b">
        <div class="flex flex-wrap instance-tag">
       
          <!-- real tags -->
          <a-tag v-for="tag in instanceInfo?.config.tag" :key="tag" class="tag" color="blue">
            {{ tag }}
          </a-tag>
        </div>
      </div>
    </a-typography-paragraph>

    <a-typography-paragraph v-if="instanceGameServerInfo">
      <div class="info-a">{{ t("TXT_CODE_855c4a1c") }}</div>
      <div class="info-b">{{ instanceGameServerInfo.players }}</div>
    </a-typography-paragraph>

    <a-typography-paragraph v-if="instanceGameServerInfo">
      <div class="info-a">{{ t("TXT_CODE_e260a220") }}</div>
      <div class="info-b">{{ instanceGameServerInfo.version }}</div>
    </a-typography-paragraph>

    <a-typography-paragraph>
      <div class="info-a">{{ t("TXT_CODE_46f575ae") }}</div>
      <div class="info-b">{{ parseTimestamp(instanceInfo?.config.lastDatetime) }}</div>
    </a-typography-paragraph>

    <a-typography-paragraph v-if="instanceInfo?.config.processType === 'docker'">
      <div class="info-a">{{ t("TXT_CODE_4f917a65") }}</div>
      <div class="info-b">
        <a href="javascript:;" @click="DockerInfoDialog?.openDialog()">
          {{ t("TXT_CODE_530f5951") }}
        </a>
      </div>
    </a-typography-paragraph>

    <a-typography-paragraph v-if="!instanceGameServerInfo">
      <div class="info-a">{{ t("TXT_CODE_8b8e08a6") }}</div>
      <div class="info-b">{{ parseTimestamp(instanceInfo?.config.createDatetime) }}</div>
    </a-typography-paragraph>

    <a-typography-paragraph v-if="!instanceGameServerInfo">
      <div class="info-a">
        <span>{{ t("TXT_CODE_cec321b4") }}{{ instanceInfo?.config.oe.toUpperCase() }}</span>
      </div>
      <div class="info-b">
        <span>{{ t("TXT_CODE_400a4210") }}{{ instanceInfo?.config.ie.toUpperCase() }}</span>
      </div>
    </a-typography-paragraph>
  </template>
</CardPanel>
  

  <CardPanel class="containerWrapper" style="height: 100%">
    <!-- 卡片标题区域 -->
    <template #title>
      实例监控
    </template>

    <!-- 卡片内容区域 -->
    <template #body>
      <!-- 顶部：左侧标题 + 右侧AI操作 -->
      <div class="flex justify-between items-center" style="margin-bottom: 16px;">
        <h3 style="margin: 0;">实例监控</h3>
        <div style="cursor: pointer; color: #0052d9;">
          试试用AI分析实例监控
        </div>
      </div>

      <!-- 四块监控项，2行2列布局 -->
      <t-row :gutter="[16, 16]">
        <!-- CPU利用率 -->
        <t-col :span="12">
          <t-card  style="height: 214px;">
            <p>当前：2.683% 总量：2核</p>
            <div style="height: 80px; margin-top: 8px;">
              <!-- 这里示例性用 t-chart，需要您配置 chartData / chartOptions -->
              <t-chart />
            </div>
          </t-card>
        </t-col>

        <!-- 内存使用量 -->
        <t-col :span="12">
          <t-card  style="height: 214px;">
            <p>当前：1254MB 总量：4GB</p>
            <div style="height: 80px; margin-top: 8px;">
              <t-chart  />
            </div>
          </t-card>
        </t-col>

        <!-- 公网带宽使用 -->
        <t-col :span="12">
          <t-card  style="height: 214px;">
            <p>当前：0.0008（入） 0.001（出）</p>
            <div style="height: 80px; margin-top: 8px;">
              <t-chart  />
            </div>
          </t-card>
        </t-col>

        <!-- 系统盘IO -->
        <t-col :span="12">
          <t-card  style="height: 214px;">
            <p>当前：0（读） 61.811（写）</p>
            <div style="height: 80px; margin-top: 8px;">
              <t-chart />
            </div>
          </t-card>
        </t-col>
      </t-row>
    </template>
  </CardPanel>

  <CardPanel class="containerWrapper" style="height: 100%">
    <template #title>
     应用信息
    </template>
    <template #body>
      <a-typography-paragraph>
        <a-typography-text :title="instanceInfo?.instanceUuid">
          <div class="info-a"> 游戏名称</div>
        </a-typography-text>
        <div class="info-b"><a-typography-text :copyable="{ text: instanceInfo?.instanceUuid }"> </a-typography-text>
        </div>
      </a-typography-paragraph>

     
    </template>
  </CardPanel>
  <t-card style="height: 100%;">
    <!-- 标题插槽 -->
    <template #title>
      <!-- 让标题和“查看更多”在同一行的布局 -->
      <div style="display: flex; justify-content: space-between; align-items: center;">
        <h3 style="margin: 0;">了解更多</h3>

        <!-- 右侧“查看更多”链接，带外链图标 -->
        <t-link
          href="https://cloud.tencent.com/document/product/1207/44569"
          target="_blank"
          theme="primary"
        >
          查看更多
          <!-- 如果项目中已引入 TDesign 图标，可以用 <t-icon>，也可用内置 icon 名 -->
          <!-- 这里仅示例使用 name="jump" 的图标，需按需引入 -->
          <t-icon name="jump" />
        </t-link>
      </div>
    </template>

    <!-- 内容插槽 -->
    <template #default>
      <!-- 使用 TDesign 栅格布局，将 4 条链接分为两列（每列占 12，共 24 栅格） -->
      <t-row :gutter="[0, 8]">
        <t-col :span="12">
          <t-link
            href="https://cloud.tencent.com/document/product/1207/53135"
            target="_blank"
            theme="default"
          >
            如何上传文件到轻量应用服务器？
          </t-link>
        </t-col>
        <t-col :span="12">
          <t-link
            href="https://cloud.tencent.com/document/product/1207/49819"
            target="_blank"
            theme="default"
          >
            轻量应用服务器相比云服务器CVM有何使用限制？
          </t-link>
        </t-col>
        <t-col :span="12">
          <t-link
            href="https://cloud.tencent.com/document/product/1207/50103"
            target="_blank"
            theme="default"
          >
            轻量应用服务器与其他云产品能否内网互通？
          </t-link>
        </t-col>
        <t-col :span="12">
          <t-link
            href="https://cloud.tencent.com/document/product/1207/48546"
            target="_blank"
            theme="default"
          >
            如何对轻量应用服务器进行备份？
          </t-link>
        </t-col>
      </t-row>
    </template>
  </t-card>


  
  <DockerInfo ref="DockerInfoDialog" :docker-info="instanceInfo?.config.docker" />
</div>
</template>

<style lang="scss" scoped>
.statuscircle {
  margin-left: 4px;
  transform: translateY(-2px);
}
.ant-typography{
  display: table-row;
}
.containerWrapper{
  display: table;
  min-height: 383px;
}
.cont {
 align-items: center;
 justify-content: space-between;
  display: flex;
}
.instance-tag {
  margin-left: -4px;
  margin-right: -4px;
  .tag {
    margin: 4px;
  }
}

.info-a{

  display: inline-flex
;
    align-items: center;
    white-space: nowrap;
    font-size: var(12px);
    transform: translateZ(0);
    vertical-align: baseline;
    color: #00000080;
    padding-right: 20px;
    padding-top: calc(30px/ 2 - 9px);
    padding-bottom: calc(30px/ 2 - 9px);
  
}
.info-b{
  padding: 6px 24px 16px 0px;
  display: table-cell;
    vertical-align: top;
    padding-bottom: 16px;
    word-wrap: break-word;
    word-break: break-word;
    padding-right: 24px;
    width: 100%;
    padding-top: calc(30px/ 2 - 9px);
}
.flex {
  display: flex;
}
.justify-between {
  justify-content: space-between;
}
.items-center {
  align-items: center;
}
</style>
<script>

</script>