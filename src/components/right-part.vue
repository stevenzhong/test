<template>
  <div :class="['rate-plan-right-part', disabled && 'is-disabled']">
    <p>Selected Rate Plan</p>
    <div class="rate-item" :class="{'no-data': rate.list.length === 0}" v-for="rate in bakData" :key="rate.name">
      <h5 v-if="rate.list.length">{{ rate.name }}</h5>
      <div v-for="(item, index) in rate.list" :key="index">
        <p v-if="item.children.length">
          {{ item.name}}
          <a-icon
            type="delete"
            v-if="!disabled"
            @click="deletePlan({ hotelId: rate.hotelId, index, key: item.key }, '2')"
          />
        </p>
        <div class="plan-list">
          <span v-for="(plan, i) in item.children" :key="plan.value">
            {{ plan.label }}
            <a-icon
              type="close"
              v-if="!disabled"
              @click="deletePlan({ hotelId: rate.hotelId, index, key: item.key, planIndex: i },'1')"
            />
          </span>
        </div>
      </div>
    </div>
  </div>
</template>
<script lang="ts">
import { Vue, Component, Prop, Watch } from 'vue-property-decorator';

@Component
export default class RightPart extends Vue {
  @Prop({ type: Object, default: () => ({}) }) public ratePlanData!: any;
  @Prop({ type: Boolean, default: false }) public disabled!: boolean;
  public bakData: any = {};

  @Watch('ratePlanData.checkedKeys', { immediate: true, deep: true })
  public checkedChange() {
    this.init();
  }

  public init() {
    const obj: any = {};
    if (
      this.ratePlanData.checkedKeys &&
      Object.keys(this.ratePlanData.checkedKeys)
    ) {
      const data = JSON.parse(JSON.stringify(this.ratePlanData.checkedKeys));
      const checkedData = this.ratePlanData.checkedData;
      for (const key in data) {
        const hotelId = key.split('-')[0];
        obj[hotelId] = obj[hotelId] || {};
        obj[hotelId].hotelId = '' + hotelId;
        obj[hotelId].name = obj[hotelId].name || checkedData[hotelId].label;
        obj[hotelId].list = obj[hotelId].list || [];
        if (data[key] && data[key].length) {
          obj[hotelId].list.push({
            name: checkedData[key].label,
            children: data[key],
            key,
          });
        }
      }
      this.bakData = JSON.parse(JSON.stringify(obj));
    }
  }

  public deletePlan(value: any, type: string) {
    const { hotelId, index, planIndex, key } = value;
    if (type === '2') {
      this.bakData[hotelId].list[index].children = [];
      this.$emit('change', { key, list: [] });
    } else {
      this.bakData[hotelId].list[index].children.splice(planIndex, 1);
      const list = this.bakData[hotelId].list[index].children.map(
        (child: any) => child.value,
      );
      this.$emit('change', { key, list });
    }
    if (this.bakData[hotelId].list[index].children.length === 0) {
      this.bakData[hotelId].list.splice(index, 1);
    }
    if (this.bakData[hotelId].list.length === 0) {
      delete this.bakData[hotelId];
    }
  }
}
</script>
<style lang="scss">
.rate-plan-right-part {
  width: 538px;
  max-height: 500px;
  overflow-y: auto;
  flex-shrink: 0;
  padding: 16px;
  border: 1px solid #d3dce6;
  box-sizing: border-box;
  box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.12), 0px 0px 6px rgba(0, 0, 0, 0.04);
  border-radius: 4px;
  margin-bottom: 40px;
  > *:last-child {
    margin-bottom: 0;
  }
  > p {
    font-size: 14px;
    line-height: 17px;
    color: #000000;
    margin-bottom: 20px;
  }
  &.is-disabled{
    .rate-item{
      > div{
        .plan-list{
          > span{
            background: none;
            color: #000000;
            border: none;
            padding: 0;
            margin-bottom: 12px;
            margin-right: 18px;
          }
        }
      }
    }
  }
  .rate-item {
    border-bottom: 1px solid rgba(0, 0, 0, 0.09);
    margin-bottom: 20px;
    &.no-data{
      margin-bottom: 0px;
      border-bottom: none;
    }
    &:last-child{
      border-bottom: none;
      margin-bottom: 0px;
    }
    > h5 {
      font-size: 16px;
      line-height: 19px;
      color: #000000;
      font-weight: bold;
      margin-bottom: 12px;
    }
    > div {
      > p {
        font-size: 14px;
        line-height: 17px;
        color: #000000;
        margin-bottom: 12px;
        display: flex;
        justify-content: space-between;
        > i {
          margin-left: 12px;
          cursor: pointer;
        }
      }
      .plan-list {
        display: flex;
        flex-wrap: wrap;
        > span {
          padding: 5px 8px;
          background: rgba(0, 145, 255, 0.05);
          border: 1px solid #0091ff;
          border-radius: 4px;
          font-size: 12px;
          line-height: 16px;
          color: #0091ff;
          margin-bottom: 16px;
          margin-right: 8px;
          > i {
            margin-left: 4px;
            cursor: pointer;
          }
        }
      }
    }
  }
}
</style>
