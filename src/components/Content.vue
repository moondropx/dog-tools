<template>
  <a-layout-content class="content">
    <slot />
  </a-layout-content>
</template>

<script lang="ts">
import Vue from 'vue';
import { Component } from 'vue-property-decorator';

@Component
export default class Content extends Vue {}
</script>

<style lang="less" scoped>
.content {
  margin: 30px;
}
</style>
