<template>
  <Content>
    <h3>设置</h3>
    <label>
      秒数
      <a-input-number v-model="timeUpper" @change="renderGraph" :min="10" />
    </label>
    <label style="margin-left: 8px">
      CDR
      <a-input-number v-model="cdr" @change="renderGraph" :min="0" :max="100" />
    </label>
    <h3>预设</h3>
    <a-checkbox-group
      v-model="checkedList"
      :options="presets.map(preset => preset.name)"
      @change="renderGraph"
    />
    <!-- <h3>自定义</h3>
    <span>
      <a-checkbox />
      <label>初动 <a-input-number /></label>
      <label>再动 <a-input-number /></label>
      <label>持续 <a-input-number /></label>
      <label>持续 <a-input-number /></label>
    </span> -->
    <!-- <a-button type="dashed">增加</a-button> -->
    <div id="chart" />
  </Content>
</template>

<script lang="ts">
import Vue from 'vue';
import { Component } from 'vue-property-decorator';
import { Chart } from '@antv/g2';
import _ from 'lodash';
import Content from '@/components/Content.vue';

type SkillType = {
  /** 名称 */
  name: string;
  /** 初动 */
  ct: number;
  /** 再动 */
  wt: number;
  /** 持续 */
  last: number;
  /** 单次回的c */
  cp: number;
  /** 持续型的多久一回 */
  pt: number;
  /** 自带减cd */
  cdr?: number;
  /** 变技能 */
  change?: {
    wt: number;
    last: number;
    cp: number;
    pt: number;
  };
};

const presets: SkillType[] = [
  {
    name: '自然回c',
    ct: 0,
    wt: 0,
    last: 0,
    cp: 0,
    pt: 55,
  },
  {
    name: '技觉黑兵',
    ct: 5 * 60,
    wt: 0,
    last: 0,
    cp: 10,
    pt: 120,
  },
  {
    name: '技觉TTN',
    ct: 15 * 60,
    wt: 30 * 60,
    last: 10 * 60,
    cp: 0,
    pt: 28,
  },
  {
    name: '技觉黑商',
    ct: 5 * 60,
    wt: 30 * 60,
    last: 15 * 60,
    cp: 0,
    pt: 28,
    change: {
      wt: 30 * 60,
      last: 30 * 60,
      cp: 0,
      pt: 60,
    },
  },
  {
    name: '白援军3',
    ct: (35 * 60) / 2,
    wt: 35 * 60,
    last: 2 * 60,
    cp: 15,
    pt: 0,
  },
  {
    name: '技觉白肉',
    ct: (20 * 60) / 2,
    wt: 0,
    last: 40 * 60,
    cp: 0,
    pt: 60,
  },
  {
    name: '花嫁黑商',
    ct: 1 * 60,
    wt: 35 * 60,
    last: 2 * 60,
    cp: 10,
    pt: 0,
    cdr: 0.45,
  },
];

@Component({ components: { Content } })
export default class Cost extends Vue {
  private presets = presets;
  private checkedList: string[] = ['自然回c'];
  private timeUpper = 60;
  private cdr = 0;
  private chart: Chart | null = null;

  getCostData(prof: SkillType, frameUpper: number, cdr?: number) {
    let frame = 0;
    let first = 2;
    let cost = 0;
    const frames: Array<{ frame: number; cost: number; name: string }> = [
      { frame: 0, cost, name: prof.name },
    ];
    while (frame <= frameUpper) {
      if (first === 2) {
        frame += prof.ct;
        first = 1;
      } else {
        // 再动为0的技能为单次技能
        if (prof.wt === 0) {
          break;
        }

        frame += prof.wt * (1 - Math.max(cdr || 0, prof.cdr || 0));
        if (first === 1) {
          if (prof.change) {
            prof = { ...prof, ...prof.change };
          }
          first = 0;
        }
      }
      if (frame <= frameUpper) {
        if (!prof.pt) {
          cost += prof.cp;
          frames.push({ frame, cost, name: prof.name });
          frame += prof.last;
        } else {
          // 开技回一点c的永续
          if (prof.cp) {
            cost += prof.cp;
            frames.push({ frame, cost, name: prof.name });
          }

          for (
            let g = prof.pt;
            (g < prof.last || prof.last === 0) && frame + g <= frameUpper;
            g += prof.pt
          ) {
            frames.push({ frame: frame + g, cost: ++cost, name: prof.name });
          }
          // 永续的直接跳出
          if (prof.last === 0) {
            break;
          }
          frame += prof.last;
        }
      }
    }
    frames.push({ frame: frameUpper, cost, name: prof.name });
    return frames.map(frame => ({ ...frame, time: frame.frame / 60 }));
  }

  renderGraph() {
    const data = _.flatten(
      this.checkedList
        .map(name => this.presets.find(p => p.name === name))
        .filter(p => p)
        .map(preset =>
          // eslint-disable-next-line
          this.getCostData(preset!, this.timeUpper * 60, this.cdr / 100),
        ),
    );

    if (this.chart) {
      this.chart.data(data);

      this.chart.render();
    }
  }

  mounted() {
    this.chart = new Chart({
      container: 'chart', // 指定图表容器 ID
      width: 800, // 指定图表宽度
      height: 300, // 指定图表高度
    });
    this.chart.scale({
      time: { min: 0, alias: '秒数' },
      cost: { min: 0, alias: 'cost', sync: true },
    });

    this.chart.axis('time', { title: {} });
    this.chart.axis('cost', { title: {} });

    this.chart
      .line()
      .position('time*cost')
      .shape('hv')
      .color('name');
    this.renderGraph();
  }
}
</script>

<style lang="less" scoped>
h3 {
  margin-top: 10px;
}
</style>
