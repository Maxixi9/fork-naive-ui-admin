<template>
  <NMenu
    :options="menus"
    :inverted="inverted"
    :mode="mode"
    :collapsed="collapsed"
    :collapsed-width="64"
    :collapsed-icon-size="20"
    :indent="24"
    :expanded-keys="openKeys"
    :value="getSelectedKeys"
    @update:value="clickMenuItem"
    @update:expanded-keys="menuExpanded"
  />
</template>

<script lang="ts">
  import { defineComponent, ref, onMounted, reactive, computed, watch, toRefs, unref } from 'vue';
  import { useRoute, useRouter } from 'vue-router';
  import { useAsyncRouteStore } from '@/store/modules/asyncRoute';
  import { generatorMenu, generatorMenuMix } from '@/utils';
  import { useProjectSettingStore } from '@/store/modules/projectSetting';
  import { useProjectSetting } from '@/hooks/setting/useProjectSetting';

  export default defineComponent({
    name: 'Menu',
    components: {},
    props: {
      mode: {
        // 菜单模式
        type: String,
        default: 'vertical',
      },
      collapsed: {
        // 侧边栏菜单是否收起
        type: Boolean,
      },
      //位置
      location: {
        type: String,
        default: 'left',
      },
    },
    emits: ['update:collapsed', 'clickMenuItem'],
    setup(props, { emit }) {
      // 当前路由
      const currentRoute = useRoute();
      const router = useRouter();
      const asyncRouteStore = useAsyncRouteStore();
      const settingStore = useProjectSettingStore();
      const menus = ref<any[]>([]);
      const selectedKeys = ref<string>(currentRoute.name as string);
      const headerMenuSelectKey = ref<string>('');

      const { getNavMode } = useProjectSetting();

      const navMode = getNavMode;

      // 获取当前打开的子菜单
      const matched = currentRoute.matched;

      const getOpenKeys = matched && matched.length ? matched.map((item) => item.name) : [];

      const state = reactive({
        openKeys: getOpenKeys,
      });

      const inverted = computed(() => {
        return ['dark', 'header-dark'].includes(settingStore.navTheme);
      });

      const getSelectedKeys = computed(() => {
        let location = props.location;
        return location === 'left' || (location === 'header' && unref(navMode) === 'horizontal')
          ? unref(selectedKeys)
          : unref(headerMenuSelectKey);
      });

      // 监听分割菜单
      watch(
        () => settingStore.menuSetting.mixMenu,
        () => {
          updateMenu();
          if (props.collapsed) {
            emit('update:collapsed', !props.collapsed);
          }
        }
      );

      // 监听菜单收缩状态
      // watch(
      //   () => props.collapsed,
      //   (newVal) => {
      //   }
      // );

      // 跟随页面路由变化，切换菜单选中状态
      watch(
        () => currentRoute.fullPath,
        () => {
          updateMenu();
          const matched = currentRoute.matched;
          state.openKeys = matched.map((item) => item.name);
          const activeMenu: string = (currentRoute.meta?.activeMenu as string) || '';
          selectedKeys.value = activeMenu ? (activeMenu as string) : (currentRoute.name as string);
        }
      );

      function updateMenu() {
        if (!settingStore.menuSetting.mixMenu) {
          menus.value = generatorMenu(asyncRouteStore.getMenus);
        } else {
          //混合菜单
          const firstRouteName: string = (currentRoute.matched[0].name as string) || '';
          menus.value = generatorMenuMix(asyncRouteStore.getMenus, firstRouteName, props.location);
          const activeMenu: string = currentRoute?.matched[0].meta?.activeMenu as string;
          headerMenuSelectKey.value = (activeMenu ? activeMenu : firstRouteName) || '';
        }
      }

      // 点击菜单
      function clickMenuItem(key: string) {
        if (/http(s)?:/.test(key)) {
          // 打开外部链接frame
          window.open(key);
        } else {
          // 内部路由
          router.push({ name: key });
          if (key === 'dashboard_workplace') {
            settingStore.navMode = key === 'dashboard_workplace' ? 'horizontal-mix' : 'horizontal';
            // settingStore.multiTabsSetting.show = key === 'dashboard_workplace';
          }
        }
        emit('clickMenuItem' as any, key);
      }

      //展开菜单
      function menuExpanded(openKeys: string[]) {
        if (!openKeys) return;
        const latestOpenKey = openKeys.find((key) => state.openKeys.indexOf(key) === -1);
        const isExistChildren = findChildrenLen(latestOpenKey as string);
        state.openKeys = isExistChildren ? (latestOpenKey ? [latestOpenKey] : []) : openKeys;
      }

      //查找是否存在子路由
      function findChildrenLen(key: string) {
        if (!key) return false;
        const subRouteChildren: string[] = [];
        for (const { children, key } of unref(menus)) {
          if (children && children.length) {
            subRouteChildren.push(key as string);
          }
        }
        return subRouteChildren.includes(key);
      }

      onMounted(() => {
        updateMenu();
      });

      return {
        ...toRefs(state),
        inverted,
        menus,
        selectedKeys,
        headerMenuSelectKey,
        getSelectedKeys,
        clickMenuItem,
        menuExpanded,
      };
    },
  });
</script>
<style lang="less">
  .n-menu {
    font-size: 16px;
    color: #dcdcdc;
    .n-menu-item {
      margin: 0;
      height: 60px;
      line-height: 60px;
    }
  }
  .n-submenu {
    font-size: 15px;
    .n-submenu-children {
      font-size: 14px;
      .n-menu-item-content--selected::before {
        background: #2a394f !important;
      }
      // .n-menu-item-content:hover {
      //   background: #2a394f;
      // }
      .n-menu-item-content--selected:hover {
        background: #2a394f !important;
      }
    }
  }
  .n-menu.n-menu--horizontal {
    padding: 0;
    .n-menu-item-content {
      border-right: 1px solid #28364a;
      border-left: 1px solid #43546d;
      width: 180px;
      text-align: center;
      border-bottom: 1px solid #43546d;
    }
    .n-menu-item-content--selected::before {
      background: #2a394f;
    }
    .n-menu-item-content:hover {
      background: #2a394f;
      border-bottom: 3px solid #15a4fa !important;
      .n-menu-item-content__icon {
        color: #15a4fa !important;
      }
    }
    .n-menu-item-content--child-active {
      background: #2a394f;
    }
    .n-menu-item-content--selected {
      background: #2a394f;
      border-bottom: 3px solid #15a4fa !important;
      .n-menu-item-content__icon {
        color: #15a4fa !important;
      }
    }
  }
</style>
