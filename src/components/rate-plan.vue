<template>
  <div class="hotel-rate-plan-page">
    <tree-select
      ref="treeSelect"
      :default-value="value"
      :options="options"
      :disabled="disabled"
      @change="treeSelectChange"
      :height="500">
    </tree-select>
    <right-part :rate-plan-data="ratePlanData" @change="planSelectChange" :disabled="disabled"></right-part>
  </div>
</template>
<script lang="ts">
import { Vue, Component, Model, Prop } from 'vue-property-decorator';
import TreeSelect from './tree-select.vue';
import RightPart from './right-part.vue';

@Component({
  components: {
    TreeSelect,
    RightPart,
  },
})
export default class RatePlanPage extends Vue {
  @Model('change', { type: Array, default: () => [] }) public readonly value: any;
  @Prop({ type: Boolean, default: false }) public disabled!: boolean;

  public options = [
    {
      label: '测试数据1',
      value: 1,
      children: [
        {
          label: '测试数据1-1',
          value: 1,
          children: [
            {
              label: '测试数据1-1-1',
              value: 1,
              children: [],
            },
            {
              label: '测试数据1-1-2',
              value: 2,
              children: [],
            },
            {
              label: '测试数据1-1-3',
              value: 3,
              children: [],
            },
            {
              label: '测试数据1-1-4',
              value: 4,
              children: [],
            },
            {
              label: '测试数据1-1-5',
              value: 5,
              children: [],
            },
          ],
        },
        {
          label: '测试数据1-2',
          value: 2,
          children: [
            {
              label: '测试数据1-2-1',
              value: 6,
              children: [],
            },
            {
              label: '测试数据1-2-2',
              value: 7,
              children: [],
            },
            {
              label: '测试数据1-2-3',
              value: 8,
              children: [],
            },
            {
              label: '测试数据1-2-4',
              value: 9,
              children: [],
            },
            {
              label: '测试数据1-2-5',
              value: 10,
              children: [],
            },
          ],
        },
        {
          label: '测试数据1-3',
          value: 3,
          children: [
            {
              label: '测试数据1-3-1',
              value: 11,
              children: [],
            },
            {
              label: '测试数据1-3-2',
              value: 12,
              children: [],
            },
            {
              label: '测试数据1-3-3',
              value: 13,
              children: [],
            },
            {
              label: '测试数据1-3-4',
              value: 14,
              children: [],
            },
            {
              label: '测试数据1-3-5',
              value: 15,
              children: [],
            },
          ],
        },
      ],
    },
    {
      label: '测试数据2',
      value: 2,
      children: [
        {
          label: '测试数据2-1',
          value: 1,
          children: [
            {
              label: '测试数据2-1-1',
              value: 16,
              children: [],
            },
            {
              label: '测试数据2-1-2',
              value: 17,
              children: [],
            },
            {
              label: '测试数据2-1-3',
              value: 18,
              children: [],
            },
            {
              label: '测试数据2-1-4',
              value: 19,
              children: [],
            },
            {
              label: '测试数据2-1-5',
              value: 20,
              children: [],
            },
          ],
        },
        {
          label: '测试数据2-2',
          value: 2,
          children: [
            {
              label: '测试数据2-2-1',
              value: 21,
              children: [],
            },
            {
              label: '测试数据2-2-2',
              value: 22,
              children: [],
            },
            {
              label: '测试数据2-2-3',
              value: 23,
              children: [],
            },
            {
              label: '测试数据2-2-4',
              value: 24,
              children: [],
            },
            {
              label: '测试数据2-2-5',
              value: 25,
              children: [],
            },
          ],
        },
        {
          label: '测试数据2-3',
          value: 3,
          children: [
            {
              label: '测试数据2-3-1',
              value: 26,
              children: [],
            },
            {
              label: '测试数据2-3-2',
              value: 27,
              children: [],
            },
            {
              label: '测试数据2-3-3',
              value: 28,
              children: [],
            },
            {
              label: '测试数据2-3-4',
              value: 29,
              children: [],
            },
            {
              label: '测试数据2-3-5',
              value: 30,
              children: [],
            },
          ],
        },
      ],
    },
  ];
  public ratePlanData: any = {};

  public treeSelectChange(data: any) {
    this.ratePlanData = data;
    this.$emit('change', data.data.map((v: any) => v.split('-').pop()));
  }
  public planSelectChange(data: any) {
    const { list, key } = data;
    (this.$refs.treeSelect as TreeSelect).changeValue(list, key);
  }
}
</script>
<style lang="scss">
.hotel-rate-plan-page{
  display: flex;
  flex-wrap: wrap;
  .hotel-tree-selected{
    flex-shrink: 0;
    margin-right: 40px;
    margin-bottom: 40px;
  }
}
</style>
