<template>
  <Content>
    <div>2333</div>
  </Content>
</template>

<script lang="ts">
import Vue from 'vue';
import Content from '@/components/Content.vue';
import { Component } from 'vue-property-decorator';

@Component({ components: { Content } })
export default class Cost extends Vue {}
</script>
