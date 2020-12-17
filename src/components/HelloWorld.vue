<template>
  <div class="room-list">
    <rate-plan v-model="list"></rate-plan>
  </div>
</template>

<script lang="ts">
import { Vue, Component, Watch } from 'vue-property-decorator';
import RatePlan from './rate-plan.vue';

@Component({
  components: {
    RatePlan,
  },
})
export default class Index extends Vue {
  public list = ['3', '12', '13', '25'];
}
</script>

