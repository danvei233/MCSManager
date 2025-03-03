<script setup lang="ts">
import { ref, onMounted, computed } from "vue";
import CardPanel from "@/components/CardPanel.vue";
import InstanceServerConfigOverview from "@/widgets/instance/ServerConfigOverview.vue";
import { t } from "@/lang/i18n";
import InstanceBaseInfo from "@/widgets/instance/BaseInfo.vue";
import InstanceFileManager from "@/widgets/instance/FileManager.vue";

import type { LayoutCard } from "@/types";
import {
  CloudDownloadOutlined,
  CloudServerOutlined,
  DownOutlined,
  PauseCircleOutlined,
  PlayCircleOutlined,
  RedoOutlined,
  LaptopOutlined,
  InteractionOutlined,
  LoadingOutlined,
  MoneyCollectOutlined
} from "@ant-design/icons-vue";
import { CheckCircleOutlined, InfoCircleOutlined } from "@ant-design/icons-vue";
import { arrayFilter } from "../../tools/array";
import { useTerminal } from "../../hooks/useTerminal";
import { useLayoutCardTools } from "@/hooks/useCardTools";
import { useScreen } from "@/hooks/useScreen";
import IconBtn from "@/components/IconBtn.vue";
import {
  openInstance,
  stopInstance,
  restartInstance,
  killInstance,
  updateInstance
} from "@/services/apis/instance";
import { CloseOutlined } from "@ant-design/icons-vue";
import { GLOBAL_INSTANCE_NAME } from "../../config/const";
import { INSTANCE_STATUS } from "@/types/const";
import { reportErrorMsg } from "@/tools/validator";
import TerminalCore from "@/components/TerminalCore.vue";
import Reinstall from "./dialogs/Reinstall.vue";
import { useAppStateStore } from "@/stores/useAppStateStore";
import { INSTANCE_TYPE_TRANSLATION } from "@/hooks/useInstance";
import { useMountComponent } from "@/hooks/useMountComponent";
import UseRedeemDialog from "@/components/fc/UseRedeemDialog.vue";

const props = defineProps<{
  card: LayoutCard;
}>();

const { isPhone } = useScreen();
const { state, isAdmin } = useAppStateStore();
const { getMetaOrRouteValue } = useLayoutCardTools(props.card);
const {
  execute,
  state: instanceInfo,
  isStopped,
  isRunning,
  isBuys,
  isGlobalTerminal
} = useTerminal();
const reinstallDialog = ref<InstanceType<typeof Reinstall>>();
const instanceId = getMetaOrRouteValue("instanceId");
const daemonId = getMetaOrRouteValue("daemonId");
const viewType = getMetaOrRouteValue("viewType", false);
const innerTerminalType = computed(() => props.card.width === 12 && viewType === "inner");
const instanceTypeText = computed(
  () => INSTANCE_TYPE_TRANSLATION[instanceInfo.value?.config.type ?? -1]
);

const updateCmd = computed(() => (instanceInfo.value?.config.updateCommand ? true : false));
const instanceStatusText = computed(() => INSTANCE_STATUS[instanceInfo.value?.status ?? -1]);
const quickOperations = computed(() =>
  arrayFilter([
    {
      title: t("TXT_CODE_57245e94"),
      icon: PlayCircleOutlined,
      noConfirm: false,
      type: "default",
      click: async () => {
        try {
          await openInstance().execute({
            params: {
              uuid: instanceId || "",
              daemonId: daemonId || ""
            }
          });
        } catch (error: any) {
          reportErrorMsg(error);
        }
      },
      props: {},
      condition: () => isStopped.value
    },
    {
      title: t("TXT_CODE_b1dedda3"),
      icon: PauseCircleOutlined,
      type: "default",
      click: async () => {
        try {
          await stopInstance().execute({
            params: {
              uuid: instanceId || "",
              daemonId: daemonId || ""
            }
          });
        } catch (error: any) {
          reportErrorMsg(error);
        }
      },
      props: {
        danger: true
      },
      condition: () => isRunning.value
    }
  ])
);
const instanceOperations = computed(() =>
  arrayFilter([
    {
      title: t("TXT_CODE_47dcfa5"),
      icon: RedoOutlined,
      type: "default",
      noConfirm: false,
      click: async () => {
        try {
          await restartInstance().execute({
            params: {
              uuid: instanceId || "",
              daemonId: daemonId || ""
            }
          });
        } catch (error: any) {
          reportErrorMsg(error);
        }
      },
      condition: () => isRunning.value
    },
    {
      title: t("TXT_CODE_7b67813a"),
      icon: CloseOutlined,
      type: "danger",
      click: async () => {
        try {
          await killInstance().execute({
            params: {
              uuid: instanceId || "",
              daemonId: daemonId || ""
            }
          });
        } catch (error: any) {
          reportErrorMsg(error);
        }
      },
      condition: () => !isStopped.value
    },
    {
      title: t("TXT_CODE_40ca4f2"),
      type: "default",
      icon: CloudDownloadOutlined,
      click: async () => {
        try {
          await updateInstance().execute({
            params: {
              uuid: instanceId || "",
              daemonId: daemonId || "",
              task_name: "update"
            },
            data: {
              time: new Date().getTime()
            }
          });
        } catch (error: any) {
          reportErrorMsg(error);
        }
      },
      condition: () => isStopped.value && updateCmd.value
    },
    {
      title: t("TXT_CODE_b19ed1dd"),
      icon: InteractionOutlined,
      noConfirm: true,
      click: () => reinstallDialog.value?.openDialog(),
      props: {},
      condition: () =>
        isStopped.value &&
        (state.settings.allowUsePreset || isAdmin.value) &&
        !isGlobalTerminal.value
    },
    {
      title: t("TXT_CODE_f77093c8"),
      icon: MoneyCollectOutlined,
      noConfirm: true,
      click: () => {
        useMountComponent({
          instanceId: instanceId
        }).mount(UseRedeemDialog);
      },
      props: {},
      condition: () => state.settings.businessMode
    }
  ])
);

const getInstanceName = computed(() => {
  if (instanceInfo.value?.config.nickname === GLOBAL_INSTANCE_NAME) {
    return t("TXT_CODE_5bdaf23d");
  } else {
    return instanceInfo.value?.config.nickname;
  }
});

onMounted(async () => {
  try {
    if (instanceId && daemonId) {
      await execute({
        instanceId,
        daemonId
      });
    }
  } catch (error: any) {
    console.error(error);
    throw error;
  }
});
const options = [
  { content: '重装系统', value: 1 },
  { content: '退还系统', value: 2 },
  { content: '编辑标签', value: 3 },
  { content: '升级套餐', value: 4 },
];
const clickHandler = (value: number) => {
  console.log('clickHandler', value);
};

// ===============================================================================



</script>

<template>

  <!-- i18nWCSNDMM -->
  <t-head-menu theme="light" style="padding: 0 10px; gap: 10px;">
    <t-space>
      <a style="margin-right: 10px;" href="../"><svg class="ruyi-icon ruyi-icon-arrow-left-stroke" role="img"
          aria-label="btnback" xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 16 16">
          <g fill="none">
            <path
              d="M4.27738 8.66672L7.47264 11.862L6.52984 12.8048L1.7251 8.00005L6.52984 3.19531L7.47264 4.13812L4.27738 7.33338H14.6679V8.66672H4.27738Z"
              fill="currentColor" fill-rule="evenodd" clip-rule="evenodd" fill-opacity="0.9"></path>
          </g>
        </svg>
      </a>
      <h2> {{ getInstanceName || '加载中...' }}</h2>
      <div style="background-color: #e9ecf1;">
        <span style="margin: 10px;">北京</span>
      </div>
<!-- show ip -->
 <div style="height: 50%;">
    <a-descriptions bordered size="small" >
        <a-descriptions-item size="small" label="ip">ip地址</a-descriptions-item>
     </a-descriptions>
  </div>
    </t-space>
    <div style="margin-left: auto;">
      <t-space>
        <t-button>
          <span>登录</span>
        </t-button>
        <t-button theme="default" variant="outline">
          <span>关机</span>
        </t-button>
        <t-button theme="default" variant="outline">
          <span>重启</span>
        </t-button>
        <t-button theme="default" variant="outline">
          <span>重置密码</span>
        </t-button>
        <t-button theme="default" variant="outline">
          <span>续费</span>
        </t-button>
        <t-dropdown :options="options" trigger="click" @click="clickHandler" :min-column-width="100">
          <t-button variant="outline">
            更多操作
            <template #suffix> <t-icon name="chevron-down" size="16" /></template>
          </t-button>
        </t-dropdown>
      </t-space>

    </div>
    <t-space>
      <t-space>

      </t-space>
    </t-space>
  </t-head-menu>

  <t-tabs>
    <!-- Summary Panel -->
    <t-tab-panel value="gy" label="概要" :destroyOnHide="false">
      <div style="padding: 12px; display: flex; justify-content: space-between;">
        <div style="width: 100%;">
          <InstanceBaseInfo :instance-info="instanceInfo" :card="card" />
        </div>
      </div>
    </t-tab-panel>

    <!-- Instance Details Panel -->
    <t-tab-panel value="info" label="应用详情" :destroyOnHide="false">
     
      <div class="server-management-app">
    <!-- Header notification -->
    <t-alert
      theme="info"
      message="某三方应用正在尝试访问并只开并使用"
      icon
      class="t-margin-bottom-16"
    />

    <!-- Main content -->
    <t-row :gutter="[16, 16]">
      <!-- Left panel -->
      <t-col :xs="24" :lg="18">
        <t-card title="应用管理与操作" bordered>
          <template #title>
            <t-space>
              <span>应用管理与操作</span>
              <t-badge count="1.6中文版" theme="primary" shape="round" />
            </t-space>
          </template>
          
          <template #actions>
            <t-space align="center">
              <t-avatar>土</t-avatar>
              <span>土豆帕帕</span>
            </t-space>
          </template>

          <!-- Running status -->
          <t-space direction="vertical" size="medium" style="width: 100%">
            <t-space align="center">
              <span>运行状态:</span>
              <t-tag theme="primary" variant="light">已启动</t-tag>
            </t-space>

            <!-- Action buttons -->
            <t-space>
              <t-button theme="primary" size="small">启动</t-button>
              <t-button theme="danger" size="small">停止</t-button>
              <t-button variant="outline" size="small">重启</t-button>
            </t-space>

            <!-- Operation buttons -->
            <t-space wrap>
              <t-button variant="text" size="small">导出存档</t-button>
              <t-button variant="text" size="small">定时备份存档</t-button>
              <t-button variant="text" size="small">云备份存档</t-button>
              <t-dropdown :options="importOptions">
                <t-button variant="text" size="small">导入存档 <t-icon name="chevron-down" /></t-button>
              </t-dropdown>
              <t-button variant="text" size="small">更新游戏</t-button>
              <t-button variant="text" size="small">设置运行插件</t-button>
              <t-button variant="text" size="small">设置Swap</t-button>
            </t-space>

            <!-- Configuration section -->
            <t-divider />
            <t-space align="center" justify="space-between" style="width: 100%">
              <t-typography.Title level="h5">应用配置</t-typography.Title>
              <t-button theme="primary" variant="text" size="small">
                <t-icon name="edit" />调整参数
              </t-button>
            </t-space>

            <!-- Parameters table -->
            <t-table
              :data="configData"
              :columns="configColumns"
              size="small"
              :pagination="{ pageSize: 10 }"
              stripe
              bordered
              style="width: 100%"
            />
          </t-space>
        </t-card>
      </t-col>

      <!-- Right panel -->
      <t-col :xs="24" :lg="6">
        <t-card bordered>
          <t-space direction="vertical" size="medium" style="width: 100%">
            <t-space>
              <t-icon name="help-circle" size="large" style="color: #0052d9" />
              <t-typography.Title level="h5" style="margin: 0">帕帕小助手</t-typography.Title>
            </t-space>
            
            <t-typography.Paragraph theme="secondary" style="font-size: 12px">
              由集三方应用插件提供技术支持，请注意遵循相应规范
            </t-typography.Paragraph>

            <!-- Question input -->
            <t-card bordered>
              <t-space direction="vertical" size="small" style="width: 100%">
                <t-space>
                  <t-icon name="help" />
                  <span>您想了解什么问题?</span>
                </t-space>
                
                <t-input placeholder="请输入您的问题，按Enter搜索" />
                
                <t-space style="color: #999; font-size: 12px">
                  <t-icon name="time" />
                  <span>历史记录:</span>
                  <span>今日已解决: 0</span>
                  <span>昨日可解决: 0</span>
                </t-space>
              </t-space>
            </t-card>

            <!-- FAQ section -->
            <t-space direction="vertical" style="width: 100%">
              <t-button 
                v-for="(faq, index) in faqs" 
                :key="index" 
                variant="text" 
                block 
                align="left"
              >
                {{ faq }}
              </t-button>
            </t-space>

            <!-- Common tutorials -->
            <t-divider />
            <t-typography.Title level="h5">常用教程</t-typography.Title>
            <t-space direction="vertical" style="width: 100%">
              <t-button 
                v-for="(tutorial, index) in tutorials" 
                :key="index" 
                variant="text" 
                block 
                align="left"
                suffix="chevron-right"
              >
                {{ tutorial }}
              </t-button>
            </t-space>
          </t-space>
        </t-card>
      </t-col>
    </t-row>
  </div>
     
     
      <div style="padding: 25px">
        <a-descriptions bordered>
          <a-descriptions-item label="实例ID">{{ instanceId }}</a-descriptions-item>
          <a-descriptions-item label="守护进程ID">{{ daemonId }}</a-descriptions-item>
        </a-descriptions>
      </div>
    </t-tab-panel>

    <!-- Terminal Panel -->
    <t-tab-panel value="terminal" label="终端" :destroyOnHide="false">
      <TerminalCore v-if="instanceId && daemonId" :instance-id="instanceId" :daemon-id="daemonId"
        :height="card.height" />
    </t-tab-panel>

    <!-- Game Config Panel -->
    <t-tab-panel value="game" label="游戏配置" :destroyOnHide="false">
      <!-- 游戏配置面板 -->
      <!-- <InstanceServerConfigOverview :instance-info="instanceInfo" :card="card" /> -->
    </t-tab-panel>

    <!-- File Management Panel -->
    <t-tab-panel value="file" label="文件管理" :destroyOnHide="false">
      <InstanceFileManager :instance-info="instanceInfo" :card="card" />
    </t-tab-panel>

    <!-- Settings Panel -->
    <t-tab-panel value="setting" label="设置" :destroyOnHide="false">
      <div style="padding: 25px">
        <a-list bordered>

        </a-list>
      </div>
    </t-tab-panel>
  </t-tabs>

  <!-- Terminal Page View -->
  <div v-if="innerTerminalType">
    <div class="mb-24">
      <BetweenMenus>
        <template #left>
          <div class="align-center">
            <a-typography-title class="mb-0 mr-12" :level="4">
              <CloudServerOutlined />
              <span class="ml-6"> {{ getInstanceName }} </span>
            </a-typography-title>
            <a-typography-paragraph v-if="!isPhone" class="mb-0 ml-4">
              <span class="ml-6">
                <a-tag v-if="isRunning" color="green">
                  <CheckCircleOutlined />
                  {{ instanceStatusText }}
                </a-tag>
                <a-tag v-else-if="isBuys" color="red">
                  <LoadingOutlined />
                  {{ instanceStatusText }}
                </a-tag>
                <a-tag v-else>
                  <InfoCircleOutlined />
                  {{ instanceStatusText }}
                </a-tag>
              </span>

              <a-tag color="purple"> {{ instanceTypeText }} </a-tag>

              <span v-if="instanceInfo?.watcher && instanceInfo?.watcher > 1 && !isPhone" class="ml-16">
                <a-tooltip>
                  <template #title>
                    {{ $t("TXT_CODE_4a37ec9c") }}
                  </template>
                  <LaptopOutlined />
                </a-tooltip>
                <span class="ml-6" style="opacity: 0.8">
                  {{ instanceInfo?.watcher }}
                </span>
              </span>
            </a-typography-paragraph>
          </div>
        </template>
        <template #right>
          <div v-if="!isPhone">
            <template v-for="item in [...quickOperations, ...instanceOperations]" :key="item.title">
              <a-button v-if="item.noConfirm" class="ml-8" :danger="item.type === 'danger'" @click="item.click">
                <component :is="item.icon" />
                {{ item.title }}
              </a-button>
              <a-popconfirm v-else :key="item.title" :title="t('TXT_CODE_276756b2')" @confirm="item.click">
                <a-button class="ml-8" :danger="item.type === 'danger'">
                  <component :is="item.icon" />
                  {{ item.title }}
                </a-button>
              </a-popconfirm>
            </template>
          </div>

          <a-dropdown v-else>
            <template #overlay>
              <a-menu>
                <a-menu-item v-for="item in [...quickOperations, ...instanceOperations]" :key="item.title"
                  @click="item.click">
                  <component :is="item.icon" />
                  {{ item.title }}
                </a-menu-item>
              </a-menu>
            </template>
            <a-button type="primary">
              {{ t("TXT_CODE_fe731dfc") }}
              <DownOutlined />
            </a-button>
          </a-dropdown>
        </template>
      </BetweenMenus>
    </div>

  </div>

  <!-- Other Page View -->
  <CardPanel v-else class="containerWrapper" style="height: 100%">
    <template #title>
      <CloudServerOutlined />
      <span class="ml-8"> {{ getInstanceName }} </span>
      <span class="ml-8">
        <a-tag v-if="isRunning" color="green">
          <CheckCircleOutlined />
          {{ instanceStatusText }}
        </a-tag>
        <a-tag v-else-if="isBuys" color="red">
          <LoadingOutlined />
          {{ instanceStatusText }}
        </a-tag>
        <a-tag v-else>
          <InfoCircleOutlined />
          {{ instanceStatusText }}
        </a-tag>
        <a-tag color="purple"> {{ instanceTypeText }} </a-tag>
      </span>
    </template>
    <template #operator>
      <span v-for="item in quickOperations" :key="item.title" size="default" class="mr-2" v-bind="item.props">
        <IconBtn :icon="item.icon" :title="item.title" @click="item.click"></IconBtn>
      </span>
      <a-dropdown>
        <template #overlay>
          <a-menu>
            <a-menu-item v-for="item in instanceOperations" :key="item.title" @click="item.click">
              <component :is="item.icon"></component>
              <span>&nbsp;{{ item.title }}</span>
            </a-menu-item>
          </a-menu>
        </template>
        <span size="default" type="primary">
          <IconBtn :icon="DownOutlined" :title="t('TXT_CODE_fe731dfc')"></IconBtn>
        </span>
      </a-dropdown>
    </template>
    <template #body>
      <TerminalCore v-if="instanceId && daemonId" :instance-id="instanceId" :daemon-id="daemonId"
        :height="card.height" />
    </template>
  </CardPanel>

  <Reinstall ref="reinstallDialog" :daemon-id="daemonId ?? ''" :instance-id="instanceId ?? ''" />



</template>

<style lang="scss" scoped>
.t-tab-panel {
  background-color: #f2f4f8;
}

.error-card {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  top: 0;
  z-index: 10;
  border-radius: 20px;

  display: flex;
  align-items: center;
  justify-content: center;

  .error-card-container {
    overflow: hidden;
    max-width: 440px;
    border: 1px solid var(--color-gray-6) !important;
    background-color: var(--color-gray-1);
    border-radius: 4px;
    padding: 12px;
    box-shadow: 0px 0px 2px var(--color-gray-7);
  }

  @media (max-width: 992px) {
    .error-card-container {
      max-width: 90vw !important;
    }
  }
}

.console-wrapper {
  position: relative;

  .terminal-loading {
    z-index: 12;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }

  .terminal-wrapper {
    border: 1px solid var(--card-border-color);
    position: relative;
    overflow: hidden;
    height: 100%;
    background-color: #1e1e1e;
    padding: 8px;
    border-radius: 6px;
    overflow: hidden;
    display: flex;
    flex-direction: column;

    .terminal-container {
      // min-width: 1200px;
      height: 100%;
    }

    margin-bottom: 12px;
  }

  .command-input {
    position: relative;

    .history {
      display: flex;
      max-width: 100%;
      overflow: scroll;
      z-index: 10;
      position: absolute;
      top: -35px;
      left: 0;

      li {
        list-style: none;

        span {
          padding: 3px 20px;
          max-width: 300px;
          overflow: hidden;
          text-overflow: ellipsis;
          cursor: pointer;
        }
      }

      &::-webkit-scrollbar {
        width: 0 !important;
        height: 0 !important;
      }
    }
  }
}
</style>
