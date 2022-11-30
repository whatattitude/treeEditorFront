<template>
  <div class="infinite-list-wrapper" style="overflow: auto">
    <ul
      v-infinite-scroll="load"
      class="list"
      :infinite-scroll-disabled="disabled"
    >
      <li
        v-for="i in count"
        :key="i"
        class="list-item"
        @click="showCenterDialogVisible(i)"
      >
        <a>{{ alarmListGroup[i].alarmList }}</a>

        <el-select
          v-model="value[i].dependence"
          id="RelySelect"
          clearable
          placeholder="Select"
          :key="value[i].dependence"
          @change="relayChange(i)"
        >
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          />
        </el-select>
      </li>
      <el-dialog
        v-model="centerDialogVisible"
        title="修改报警配置"
        width="30%"
        align-center
      >
        <span>报警组： {{ alarmList }}</span>
        <br />
        <a>报警配置: <br />{{ alarmConfig }}</a>
        <template #footer>
          <span class="dialog-footer">
            <el-button @click="closeCenterDialogVisible('delete')"
              >去除</el-button
            >
            <el-button type="primary" @click="closeCenterDialogVisible('add')">
              添加
            </el-button>
          </span>
        </template>
      </el-dialog>
    </ul>
    <p v-if="loading">Loading...</p>
    <p v-if="noMore">No more</p>
  </div>
</template>

<script lang="ts" setup>
import { METHODS } from "http";
import { method, range } from "lodash";
import { computed, onMounted, ref } from "vue";
var centerDialogVisible = ref(false);

var alarmList = "cluster.wise-nginx.nwise.www";
var alarmConfig = "dhagkjhduiaksuhdkujsabn\noiaudjkisajikjds\nihsdakhdijusah";

var alarmListGroup = [
  {
    alarmList: "cluster.wise-nginx.nwise.www",
    alarmConfig: "dhagkjhduiaksuhdkujsabn\noiaudjkisajikjds\nihsdakhdijusah",
    dependence: "strong",
  },
  {
    alarmList: "cluster.wise-ngin",
    alarmConfig: "dhagkjhduiaksuhdkujsabn\noiaudjkisajikjds\nihsdakhdijusah",
    dependence: "weak",
  },
  {
    alarmList: "ginx.nwise.www",
    alarmConfig: "dhagkjhduiaksuhdkujsabn\noiaudjkisajikjds\nihsdakhdijusah",
    dependence: "weak",
  },
  {
    alarmList: "cluster.ise.www",
    alarmConfig: "dhagkjhduiaksuhdkujsabn\noiaudjkisajikjds\nihsdakhdijusah",
    dependence: "strong",
  },
  {
    alarmList: "ww",
    alarmConfig: "dhagkjhduiaksuhdkujsabn\noiaudjkisajikjds\nihsdakhdijusah",
    dependence: "strong",
  },
];

const count = ref(alarmListGroup.length - 1);
const loading = ref(false);
const noMore = computed(() => count.value >= alarmListGroup.length - 1);
const disabled = computed(() => loading.value || noMore.value);
const load = () => {
  loading.value = true;
  setTimeout(() => {
    count.value += 1;
    loading.value = false;
  }, 2000);
};

const value = ref(alarmListGroup);
const options = [
  {
    value: "strong",
    label: "强",
  },
  {
    value: "weak",
    label: "弱",
  },
];

const props = defineProps({
  url: String,
  nodeId: String,
  centerDialogVisible: Boolean,
});
console.log("=-=-=-=-=", props);

const relayChange = (i) => {
  console.log("46515641ewqeaw     ", i);
  var l = alarmListGroup.length;
  console.log("461566546 ", alarmListGroup[i].dependence);
};
const showCenterDialogVisible = (i) => {
  centerDialogVisible.value = true;
  alarmConfig = alarmListGroup[i].alarmConfig;
  alarmList = alarmListGroup[i].alarmList;
  console.log(centerDialogVisible);
  return centerDialogVisible;
};
const closeCenterDialogVisible = (action) => {
  centerDialogVisible.value = false;
  console.log(centerDialogVisible);
  return centerDialogVisible;
};
</script>

<style>
#RelySelect {
  width: 10px;
}
.infinite-list-wrapper {
  height: 300px;
  text-align: center;
}
.infinite-list-wrapper .list {
  padding: 0;
  margin: 0;
  list-style: none;
}

.infinite-list-wrapper .list-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 50px;
  background: var(--el-color-danger-light-9);
  color: var(--el-color-danger);
}
.infinite-list-wrapper .list-item + .list-item {
  margin-top: 10px;
}
.dialog-footer button:first-child {
  margin-right: 10px;
}
</style>
