<template>
  <div id="app">
    <a-drawer
      class="menu-drawer"
      style="margin: 0;"
      :visible="drawerVisible"
      placement="left"
      :closable="false"
      @close="drawerVisible = false"
    >
      <a-menu mode="inline" :selectedKeys="['#' + $route.path]">
        <a-menu-item key="#/" @click="$router.push('/')">
          <span>主页</span>
        </a-menu-item>
        <a-menu-item key="#/cost" @click="$router.push('/cost')">
          <span>回c曲线模拟器</span>
        </a-menu-item>
      </a-menu>
    </a-drawer>
    <a-layout class="layout">
      <a-layout-header class="header">
        <a-icon
          class="trigger"
          :type="drawerVisible ? 'menu-unfold' : 'menu-fold'"
          @click="() => (drawerVisible = !drawerVisible)"
        />
      </a-layout-header>
      <router-view />
    </a-layout>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import { Component, Watch } from 'vue-property-decorator';

@Component
export default class App extends Vue {
  private drawerVisible = false;

  @Watch('$route.path')
  onValueChange() {
    this.drawerVisible = false;
  }
}
</script>

<style lang="less" scoped>
.layout {
  background: #fff;
}

.header {
  background: #fff;
  padding: 0;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
}

.menu-drawer /deep/ .ant-drawer-body {
  padding: 0;
}

.trigger {
  font-size: 18px;
  line-height: 64px;
  padding: 0 24px;
  cursor: pointer;
  transition: color 0.3s;
}

.trigger:hover {
  color: #1890ff;
}
</style>
