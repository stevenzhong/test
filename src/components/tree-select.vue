<template>
  <div :class="['hotel-tree-selected', disabled && 'is-disabled']">
    <ul :style="ulStyle">
      <li
        v-for="item in part1"
        :key="item.value"
        @click="itemSelect(item.value, 'current1')"
        :class="{'active': current1 == item.value}">
        <a-checkbox
          @change="onPart1Change"
          :checked="isAllChecked(item.value)"
          :indeterminate="isIndeterminate(item.value)"
          :disabled="item.disabled || disabled">
        </a-checkbox>
        <div>
          <span>{{item.label}}</span>
          <a-icon type="right" v-if="item.children.length > 0" />
        </div>
      </li>
    </ul>
    <ul :style="ulStyle">
      <li
        v-for="item in part2[current1]"
        :key="item.value"
        @click="itemSelect(item.value, 'current2')"
        :class="{'active': current2 == item.value}">
        <a-checkbox
          :indeterminate="checkedData[item.value] && checkedData[item.value].indeterminate"
          :checked="checkedData[item.value].checked || checkedData[item.value] && checkedData[item.value].list.length === item.children.length"
          @change="onPart2Change"
          :disabled="item.disabled || disabled">
        </a-checkbox>
        <div>
          <span>{{item.label}}</span>
          <a-icon type="right" v-if="item.children.length > 0" />
        </div>
      </li>
    </ul>
    <ul v-if="checkedData[current2]" :style="ulStyle">
      <a-checkbox-group :value="checkedData[current2].list" @change="changeValue" :disabled="disabled">
        <li
          v-for="item in part3[current2]"
          :key="item.value"
          @click="itemSelect(item.value, 'current3')"
          :class="{'active': current3 == item.value}">
          <a-checkbox :value="item.value" :disabled="item.disabled"></a-checkbox>
          <div>
            <span>{{item.label}}</span>
            <a-icon type="right" v-if="item.children.length > 0" />
          </div>
        </li>
      </a-checkbox-group>
    </ul>
  </div>
</template>

<script lang="ts">
import { Vue, Component, Prop } from 'vue-property-decorator';

interface OptionsObject {
  label: string;
  value: number | string;
  children: OptionsObject | [];
}

@Component
export default class TreeSelected extends Vue {
  @Prop({type: Array, default: () => [] }) public defaultValue!: any[];  // 默认值
  @Prop({type: Array, default: () => [] }) public options!: OptionsObject[];      // 渲染原始数据
  @Prop({type: Boolean, default: false }) public disabled!: boolean;    // 全部禁用
  @Prop({type: Number, default: 180 }) public width!: number;    // ul宽度
  @Prop({type: Number, default: null }) public height!: any;    // ul宽度
  @Prop({type: Number, default: 500 }) public maxHeight!: boolean;    // ul最大高度

  public checkedData: any = {};  // 记录选中及半选状态
  public list: any[] = [];  // 根据options数据格式化组件用的数据
  public part1: any[] = []; // 存储第一级数据
  public part2: any = {};   // 存储所有第二级数据
  public part3: any = {};   // 存储所有第三级数据
  public current1: string = ''; // 第一级选中
  public current2: string = ''; // 第二级选中
  public current3: string = ''; // 第三级选中
  public checkedKeys: any = {}; // 选中数据合集
  public values: any[] = []; // 根据defaultValue数据格式化用于反选

  // 根据第二级是否全选计算第一级全选状态
  get isAllChecked() {
    return (val: any) => {
      const temp = this.part2[val].filter((v: any) => {
        if (this.checkedData[v.value].checked) {
          return v;
        }
      });
      return temp.length === this.part2[val].length;
    };
  }

  // 第二级有半选或者非全部选中即第一级为半选
  get isIndeterminate() {
    return (val: any) => {
      let indeterminate = false;
      const temp = this.part2[val].filter((v: any) => {
        if (this.checkedData[v.value].checked) {
          return v;
        } else if (this.checkedData[v.value].indeterminate) {
          indeterminate = true;
        }
      });
      return indeterminate || temp.length > 0 && temp.length < this.part2[val].length;
    };
  }

  // ul样式
  get ulStyle() {
    return {
      width: this.width ? `${this.width}px` : 'auto',
      height: this.height ? `${this.height}px` : 'auto',
      maxHeight: this.maxHeight ? `${this.maxHeight}px` : '',
      overflowY: this.maxHeight ? 'auto' : 'visible',
    };
  }

  public created() {
    this.init();
  }

  // options数据格式化 value格式化为 上级value-当前value
  public init() {
    this.list = this.options.map((option: any) => {
      option.value += '';
      if (option.children.length) {
        option.children.map((sub: any) => {
          sub.value = `${option.value}-${sub.value}`;
          sub.children = sub.children.map((third: any) => {
            const value = '' + third.value;
            if (this.defaultValue.includes(value)) {
              this.values.push(`${sub.value}-${value}`);
            }
            third.value = `${sub.value}-${third.value}`;
            return third;
          });
          this.$set(this.part3, sub.value, sub.children);
          this.$set(this.checkedData, sub.value, {
            indeterminate: false,
            list: [],
            checked: false,
            label: sub.label,
          });
          return sub;
        });
        this.$set(this.part2, option.value, option.children);
        this.$set(this.checkedData, option.value, {
          indeterminate: false,
          list: [],
          checked: false,
          label: option.label,
        });
      }
      this.part1.push(option);
      return option;
    });
    this.current1 = this.list[0] && this.list[0].value || '';
    this.current2 = this.list[0] && this.list[0].children[0] && this.list[0].children[0].value || '';
    this.checkedToLayer(this.values);
    this.defaultValue.length && this.$emit('change', this.emitData());
  }

  // 数据反显
  public checkedToLayer(list: string[]) {
    if (list.length) {
      const obj: any = {};
      list.map((value: any) => {
        const rowKey = value.split('-').slice(0, 2).join('-');
        obj[rowKey] = obj[rowKey] || [];
        obj[rowKey].push(value);
      });
      for (const key in obj) {
        this.checkedData[key].list = obj[key];
        this.changeValue(obj[key], key, true);
      }
    }
  }

  public itemSelect(value: string, current: string) {
    this.$data[current] = value;
    if (current === 'current1') {
      this.current2 = this.part2[value][0] && this.part2[value][0].value || '';
    }
  }

  // 第三级数据变化
  public changeValue(val: any[], key?: any, noEmit?: boolean) {
    if (!key) {
      key = this.current2;
    }
    this.checkedData[key].list = val;
    this.checkedData[key].indeterminate = val.length > 0 && val.length < this.part3[key].length;
    this.checkedData[key].checked = val.length === this.part3[key].length;
    this.checkedKeys[key] = val.map((v: any) => {
      return this.part3[key].find((item: any) => item.value === v);
    });
    !noEmit && this.$emit('change', this.emitData());
  }

  // 二级checkbox变化
  public onPart2Change(e: any) {
    this.checkedData[this.current2].checked = e.target.checked;
    if (!e.target.checked) {
      this.checkedData[this.current2].list = [];
      this.checkedKeys[this.current2] = [];
      this.checkedData[this.current2].indeterminate = false;
    } else {
      this.checkedData[this.current2].indeterminate = false;
      this.checkedData[this.current2].list = this.part3[this.current2].map((v: any) => v.value);
      this.checkedKeys[this.current2] = this.part3[this.current2];
    }
    this.$emit('change', this.emitData());
  }

  // 一级checkbox变化
  public onPart1Change(e: any) {
    this.checkedData[this.current1].checked = e.target.checked;
    if (!e.target.checked) {
      this.part2[this.current1].map((item: any) => {
        this.checkedData[item.value].list = [];
        this.checkedKeys[item.value] = [];
        this.checkedData[item.value].indeterminate = false;
        this.checkedData[item.value].checked = false;
      });
    } else {
      this.part2[this.current1].map((item: any) => {
        this.checkedData[item.value].list = this.part3[item.value].map((v: any) => v.value);
        this.checkedData[item.value].indeterminate = false;
        this.checkedData[item.value].checked = true;
        this.checkedKeys[item.value] = this.part3[item.value];
      });
    }
    this.$emit('change', this.emitData());
  }

  public emitData() {
    const result: any = {
      data: [],
    };
    let arr: any = [];
    if (Object.keys(this.checkedKeys).length) {
      for (const key in this.checkedKeys) {
        arr = arr.concat(this.checkedKeys[key] || []);
      }
    }
    result.data = arr.map((v: any) => v.value);
    result.originData = arr;
    result.checkedData = this.checkedData;
    result.checkedKeys = this.checkedKeys;
    return result;
  }
}
</script>

<style lang="scss">
.hotel-tree-selected{
  display: inline-flex;
  border: 1px solid #D3DCE6;
  box-sizing: border-box;
  box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.12), 0px 0px 6px rgba(0, 0, 0, 0.04);
  border-radius: 4px;
  > ul{
    flex: 0 0 auto;
    border-right: 1px solid #D3DCE6;
    margin: 0;
    padding: 16px 0px;
    &:last-child{
      border-right: none;
    }
    .ant-checkbox-group{
      width: 100%;
    }
    li {
      display: flex;
      padding: 6px 16px;
      font-size: 13px;
      line-height: 18px;
      color: rgba(71,86,105,.81);
      .ant-checkbox-indeterminate .ant-checkbox-inner{
        background: #1890ff;
        border-color: #1890ff;
        &::after{
          width: 10px;
          height: 2px;
          background: #ffffff;
        }
      }
      .ant-checkbox-indeterminate.ant-checkbox-disabled .ant-checkbox-inner{
        background-color: #f5f5f5;
        &::after{
          background: #B8B8B8;
        }
      }
      &:hover{
        background: rgba(229, 233, 242, 0.508031);
      }
      &.active{
        background: rgba(229, 233, 242, 0.508031);
      }
      div{
        margin-left: 12px;
        display: flex;
        flex: 1;
        justify-content: space-between;
        align-items: center;
        cursor: pointer;
        > i{
          flex-shrink: 0;
          margin-left: 12px;
          color: rgba(102,102,102,.7);
        }
        > span{
          white-space: nowrap;
        }
      }
    }
  }
}
</style>
