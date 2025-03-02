<script setup lang="ts">
import { router, type RouterMetaInfo } from "@/config/router";
import logo from "@/assets/logo.png";
import { useLayoutContainerStore } from "@/stores/useLayoutContainerStore";
import { useRoute } from "vue-router";
import "tdesign-vue-next/es/style/index.css";
import { Notification, MessagePlugin } from "tdesign-vue-next";
import { computed, h } from "vue";
import { useAppRouters } from "@/hooks/useAppRouters";
// 替换ant-design的notification为tdesign的Notification
import {
  LogoutIcon,
  AppleIcon,
  SaveIcon,
  CloseCircleIcon,
  CloseOctagonIcon,
  RefreshIcon,
  FillColor1Icon,
  MenuUnfoldIcon,
  Filter2Icon,
  User1Icon
} from 'tdesign-icons-vue-next';

import { useScreen } from "@/hooks/useScreen";
import CardPanel from "./CardPanel.vue";
import { $t, $t as t } from "@/lang/i18n";
import { THEME, useAppConfigStore } from "@/stores/useAppConfigStore";
import { logoutUser } from "@/services/apis/index";
// 替换message为TDesign的Message
import { useAppToolsStore } from "@/stores/useAppToolsStore";
import { useAppStateStore } from "@/stores/useAppStateStore";
import { useLayoutConfigStore } from "../stores/useLayoutConfig";
// 替换Modal为TDesign的DialogPlugin
import { DialogPlugin } from "tdesign-vue-next";

const { saveGlobalLayoutConfig, resetGlobalLayoutConfig } = useLayoutConfigStore();
const { containerState, changeDesignMode } = useLayoutContainerStore();
const { getRouteParamsUrl, toPage } = useAppRouters();
const { setTheme } = useAppConfigStore();
const { state: appTools } = useAppToolsStore();
const { isAdmin, state: appState, isLogged } = useAppStateStore();
const openNewCardDialog = () => {
  containerState.showNewCardDialog = true;
};

const { execute } = logoutUser();

const handleToPage = (url: string) => {
  containerState.showPhoneMenu = false;
  toPage({
    path: url
  });
};

const route = useRoute();

const menus = computed(() => {
  return router
    .getRoutes()
    .filter((v) => {
      const metaInfo = v.meta as RouterMetaInfo;
      if (metaInfo.condition && !metaInfo.condition()) {
        return false;
      }
      if (containerState.isDesignMode) {
        return metaInfo.onlyDisplayEditMode || metaInfo.mainMenu;
      }
      if (isAdmin.value) {
        return metaInfo.mainMenu === true && metaInfo.onlyDisplayEditMode !== true;
      }

      return (
        metaInfo.mainMenu === true &&
        isLogged.value &&
        Number(appState.userInfo?.permission) >= Number(metaInfo.permission)
      );
    })
    .map((r) => {
      return {
        name: r.name,
        path: r.path,
        meta: r.meta
      };
    });
});

const breadcrumbs = computed(() => {
  const arr = [
    {
      title: t("TXT_CODE_f5b9d58f"),
      disabled: false,
      href: `.`
    }
  ];

  const queryUrl = getRouteParamsUrl();

  if (route.meta.breadcrumbs instanceof Array) {
    const meta = route.meta as RouterMetaInfo;
    meta.breadcrumbs?.forEach((v) => {
      const params = queryUrl && !v.mainMenu ? `?${queryUrl}` : "";
      if ((appState.userInfo?.permission || 0) < v.permission) return;
      arr.push({
        title: v.name,
        disabled: false,
        href: `./#${v.path}${params}`
      });
    });
  }

  arr.push({
    title: String(route.name),
    disabled: true,
    href: `./#${route.fullPath}`
  });

  return arr;
});

const appMenus = computed(() => {
  return [
    {
      title: t("TXT_CODE_8b0f8aab"),
      icon: LogoutIcon,
      click: openNewCardDialog,
      conditions: containerState.isDesignMode,
      onlyPC: true
    },
    {
      title: t("TXT_CODE_8145d82"),
      icon: SaveIcon,
      click: async () => {
        DialogPlugin.confirm({
          header: $t("TXT_CODE_d73c8510"),
          body: $t("TXT_CODE_6d9b9f22"),
          onConfirm: async () => {
            changeDesignMode(false);
            await saveGlobalLayoutConfig();
            Notification.success({
              placement: "top",
              title: t("TXT_CODE_47c35915"),
              content: t("TXT_CODE_e10c992a")
            });
            setTimeout(() => window.location.reload(), 400);
          }
        });
      },
      conditions: containerState.isDesignMode,
      onlyPC: true
    },
    {
      title: t("TXT_CODE_5b5d6f04"),
      icon: CloseOctagonIcon,
      click: async () => {
        DialogPlugin.confirm({
          header: $t("TXT_CODE_8f20c21c"),
          body: $t("TXT_CODE_9740f199"),
          onConfirm: async () => {
            window.location.reload();
          }
        });
      },
      conditions: containerState.isDesignMode,
      onlyPC: true
    },
    {
      title: t("TXT_CODE_abd2f7e1"),
      icon: RefreshIcon,
      click: async () => {
        DialogPlugin.confirm({
          header: $t("TXT_CODE_74fa2f73"),
          body: $t("TXT_CODE_f63bfe78"),
          onConfirm: async () => {
            await resetGlobalLayoutConfig();
            Notification.success({
              placement: "top",
              title: t("TXT_CODE_15c6d4eb"),
              content: t("TXT_CODE_e10c992a")
            });
            setTimeout(() => window.location.reload(), 400);
          }
        });
      },
      conditions: containerState.isDesignMode,
      onlyPC: true
    },

    {
      title: t("TXT_CODE_f591e2fa"),
      icon: FillColor1Icon,
      click: (key: string) => {
        if (key === THEME.DARK) {
          DialogPlugin.confirm({
            header: $t("TXT_CODE_9775ccb"),
            body: $t("TXT_CODE_90b2ae00"),
            onConfirm: async () => {
              setTheme(THEME.DARK);
            }
          });
        } else {
          setTheme(THEME.LIGHT);
        }
      },
      conditions: !containerState.isDesignMode,
      onlyPC: false,
      menus: [
        {
          title: $t("TXT_CODE_673eac8e"),
          value: THEME.LIGHT
        },
        {
          title: $t("TXT_CODE_5e4a370d"),
          value: THEME.DARK
        }
      ]
    },
    {
      title: t("TXT_CODE_ebd2a6a1"),
      icon: Filter2Icon,
      click: () => {
        changeDesignMode(true);
        Notification.warning({
          placement: "bottom",
          title: t("TXT_CODE_7b1adf35"),
          content: t("TXT_CODE_6b6f1d3")
        });
      },
      conditions: !containerState.isDesignMode && isAdmin.value,
      onlyPC: true
    },
    {
      title: t("TXT_CODE_8c3164c9"),
      icon: User1Icon,
      click: () => {
        appTools.showUserInfoDialog = true;
      },
      conditions: !containerState.isDesignMode && isLogged.value,
      onlyPC: false
    },
    {
      title: t("TXT_CODE_2c69ab15"),
      icon: LogoutIcon,
      click: async () => {
        DialogPlugin.confirm({
          header: $t("TXT_CODE_9654b91c"),
          onConfirm: async () => {
            try {
              await execute();
              MessagePlugin.success(t("TXT_CODE_11673d8c"));
              setTimeout(() => {
                window.location.reload();
              }, 400);
            } catch (error) {
              console.error("退出失败:", error);
              MessagePlugin.error(t("TXT_CODE_11673d8c") + "失败");
            }
          }
        });
      },
      conditions: !containerState.isDesignMode && isLogged.value,
      onlyPC: false
    }
  ];
});

const { isPhone } = useScreen();

const openPhoneMenu = (b = false) => {
  containerState.showPhoneMenu = b;
};
</script>

<template>
  <header class="app-header-wrapper">
    <div v-if="!isPhone" class="app-header-content">
      <nav class="btns">
      <a href="." style="margin-right: 12px">
      <div class="logo">
        <img :src="logo" style="height: 18px" />
      </div>
    </a>
      </nav>
      <div class="btns">
        <div v-for="item in menus" :key="item.path" class="nav-button" @click="handleToPage(item.path)">
          <span>{{ item.name }}</span>
        </div>
        <div v-for="(item, index) in appMenus" :key="index">
            <t-dropdown v-if="item.menus && item.conditions" placement="bottom">
              <div class="nav-button" style="font-size: large;">
                <component :is="item.icon"></component>
              </div>
              <template #dropdown>
                <t-dropdown-menu>
                  <t-dropdown-item 
                    v-for="m in item.menus" 
                    :key="m.value"
                    @click="() => item.click(m.value)"
                  >
                    {{ m.title }}
                  </t-dropdown-item>
                </t-dropdown-menu>
              </template>
            </t-dropdown>
            <t-tooltip v-else-if="item.conditions" placement="bottom" :content="item.title">
              <div class="nav-button" style="font-size: large;" @click="() => item.click('')">
                <component :is="item.icon"></component>
              </div>
            </t-tooltip>
          </div>
      </div>
    </div>
  </header>
  <div v-if="!isPhone" style="height: 60px"></div>

  <!-- Menus for phone -->
  <header v-if="isPhone" class="app-header-content-for-phone">
    <CardPanel class="card-panel">
      <template #body>
        <div style="display: flex; justify-content: space-between; align-items: center">
          <div style="width: 100px" class="flex">
            <t-button variant="text" :icon="MenuUnfoldIcon" size="small" @click="openPhoneMenu(true)"></t-button>
            <div v-for="(item, index) in appMenus" :key="index">
              <t-dropdown v-if="item.menus && item.conditions && !item.onlyPC" class="phone-nav-button"
                placement="bottom">
                <t-button variant="text" :icon="h(item.icon)" size="small"></t-button>
                <template #dropdown>
                  <t-dropdown-menu>
                    <t-dropdown-item 
                      v-for="m in item.menus" 
                      :key="m.value"
                      @click="() => item.click(m.value)"
                    >
                      {{ m.title }}
                    </t-dropdown-item>
                  </t-dropdown-menu>
                </template>
              </t-dropdown>
            </div>
          </div>
          <div>
            <img :src="logo" style="height: 18px" />
          </div>
          <div style="width: 100px" class="justify-end">
            <div v-for="(item, index) in appMenus" :key="index">
              <t-button v-if="item.conditions && !item.onlyPC && !item.menus" class="phone-nav-button" variant="text"
                :icon="item.icon" size="small" @click="item.click"></t-button>
            </div>
          </div>
        </div>
      </template>
    </CardPanel>
  </header>

  <t-drawer :size="500" header="MENU" placement="top" :visible="containerState.showPhoneMenu"
    @close="() => (containerState.showPhoneMenu = false)">
    <div class="phone-menu">
      <div v-for="item in menus" :key="item.path" class="phone-menu-btn" @click="handleToPage(item.path)">
        {{ item.name }}
      </div>
    </div>
  </t-drawer>

  <div class="breadcrumbs">
    <t-breadcrumb>
      <t-breadcrumb-item v-for="item in breadcrumbs" :key="item.title">
        <a v-if="!item.disabled" :href="item.href">{{ item.title }}</a>
        <span v-else>{{ item.title }}</span>
      </t-breadcrumb-item>
    </t-breadcrumb>
  </div>
</template>

<style lang="scss" scoped>
@import "@/assets/global.scss";

.phone-menu {
  .phone-menu-btn {
    padding: 16px 8px;
    border-bottom: 1px solid var(--color-gray-4);
    color: var(--color-gray-12);
  }
}

.app-header-content-for-phone {
  height: 60px;
  width: 100%;

  // display: flex;
  // justify-content: space-between;
  // align-items: center;
  // margin: 0px;
  .card-panel {
    background-color: var(--app-header-bg);
    margin-top: 8px;

    button {
      color: var(--color-always-white) !important;
    }
  }

  .phone-nav-button,
  .phone-nav-button * {
    margin: 0px 6px;
  }
}

.breadcrumbs {
  font-size: 18px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 0px;
}

.app-header-wrapper {
  box-shadow: 0 2px 4px 0 var(--card-shadow-color);

  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: var(--app-header-bg);
  backdrop-filter: saturate(180%) blur(20px);
  color: var(--app-header-text-color);

  position: fixed;
  top: 0;
  left: 0;
  right: 0;

  z-index: 20;

  .app-header-content {
    @extend .global-app-container;
    color: #202020;
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
    height: 60px;

    .btns {
      height: 50%;
      display: flex;
      align-items: center;
    }
  }

  .btns div {
    height: 100%;
  }

  .nav-button {
    display: flex;
    align-items: center;
    height: 100%;
    text-align: center !important;
    font-size: 14px;
    transition: all 0.4s;
    color: #ffffff8c;
    text-align: center;
    padding: 8px 12px;
    min-width: 40px;
    cursor: pointer;
  }

  .icon-button {
    font-size: 16px !important;
  }

  .nav-button:hover {
    background-color: rgba(215, 215, 215, 0.12);
  }

  .logo {
    cursor: pointer;
  }

  // Sync margin
  @media (max-width: 1470px) {
    .app-header-content,
    .app-header-content-for-phone {
      margin: 0px 25px;
    }
  }

  @media (max-width: 992px) {
    .app-header-content {
      margin: 0px 8px;
    }
  }
}
</style>
