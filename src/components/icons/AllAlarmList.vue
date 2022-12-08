<template>
  <div class="infinite-list-wrapper" style="overflow: auto">
    <ul
      v-infinite-scroll="load"
      class="list"
      :infinite-scroll-disabled="disabled"
    >
      <li
        v-for="i in itemsCount"
        :key="i"
        class="list-item"
        @click="showCenterDialogVisible(i)"
      >
        <a>{{ alarmListGroup?.[i]?.alarmList }}</a>

        <el-select
          v-if="alarmListGroup[i]"
          v-model="alarmListGroup[i].dependence"
          id="RelySelect"
          clearable
          placeholder="Select"
          :key="alarmListGroup[i].dependence"
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
        <a>报警可信度: {{ alarmReliability }}</a>
        <br />
        <a>报警配置: <br />{{ alarmConfig }}</a>
        <br />

        <template #footer>
          <span class="dialog-footer">
            <el-button @click="closeCenterDialogVisible('delete')"
              >取消</el-button
            >
            <el-button
              type="primary"
              @click="closeCenterDialogVisible('connect')"
            >
              关联到当前
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
import axios from "axios";
import { reactive } from "vue";
const list = reactive([1, 2, 3]);
const ikey = 0;
var centerDialogVisible = ref(false);

var alarmList = "cluster.wise-nginx.nwise.www";
var alarmConfig = "dhagkjhduiaksuhdkujsabn\noiaudjkisajikjds\nihsdakhdijusah";
var alarmId = -1;

var alarmListGroup = ref([
  {
    alarmId: -1,
    alarmList: "cluster.wise-nginx.nwise.www",
    alarmConfig: "dhagkjhduiaksuhdkujsabn\noiaudjkisajikjds\nihsdakhdijusah",
    dependence: "weak",
    reliability: "unreliable",
  },
  {
    alarmId: 1,
    alarmList: "cluster.wise-nginx.nwise.www",
    alarmConfig: "dhagkjhduiaksuhdkujsabn\noiaudjkisajikjds\nihsdakhdijusah",
    dependence: "weak",
    reliability: "unreliable",
  },
  {
    alarmId: 1,
    alarmList: "ww",
    alarmConfig: "dhagkjhduiaksuhdkujsabn\noiaudjkisajikjds\nihsdakhdijusah",
    dependence: "strong",
    reliability: "unreliable",
  },
]);

var alarmReliability = "unreliable";
const count = ref(5);
const loading = ref(false);
const noMore = computed(() => count.value >= alarmListGroup.value.length - 1);
const disabled = computed(() => loading.value || noMore.value);
const load = () => {
  loading.value = true;
  setTimeout(() => {
    count.value += 10;

    loading.value = false;
  }, 2000);
};

const itemsCount = computed(() => {
  return count.value > alarmListGroup.value.length - 1
    ? alarmListGroup.value.length - 1
    : count.value;
});

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

const getDataAllAlarm = (url) => {
  console.log("12u3qhuidixahsxnkjahx   kla;djklas dl oaisd lkasdjasdnlksdj");
  var urlQuest = url;

  axios
    .get(urlQuest, {
      params: {},
    })
    .then(function (response) {
      alarmListGroup.value = [
        {
          alarmId: -1,
          alarmConfig: "",
          alarmList: "",
          dependence: "strong",
          reliability: "unreliable",
        },
      ];
      for (var i in response.data.Data) {
        var item = {
          alarmId: -1,
          alarmList: "",
          alarmConfig: "",
          dependence: "strong",
          reliability: "unreliable",
        };
        console.log(" ====== ---- ======", response.data.Data[i]);
        item.alarmList = response.data.Data[i].alertTernary;
        item.alarmConfig = response.data.Data[i].alertConfig;
        item.dependence = "弱";
        item.reliability = response.data.Data[i].reliability;
        item.alarmId = response.data.Data[i].id;
        alarmListGroup.value.push(item);
      }
    })
    .catch(function (error) {
      console.log(error);
    });
  console.log(" adada ", alarmListGroup);
};

const setParams = (id) => {
  nodeId = id;
};

defineExpose({
  getDataAllAlarm,
  setParams,
});

var commitUrl = "";

var nodeId = "-1";

const props = defineProps({
  url: String,
  commitUrl: String,
  nodeId: String,
  centerDialogVisible: Boolean,
});
console.log("=-=-=-=-=", props);

const relayChange = (i) => {
  console.log("46515641ewqeaw     ", i);
  var l = alarmListGroup.value.length;

  console.log("461566546 ", alarmListGroup.value[i].dependence);
};
const showCenterDialogVisible = (i) => {
  centerDialogVisible.value = true;
  alarmConfig = alarmListGroup.value[i].alarmConfig;
  alarmList = alarmListGroup.value[i].alarmList;
  alarmId = alarmListGroup.value[i].alarmId;
  alarmReliability = alarmListGroup.value[i].reliability;
  console.log(centerDialogVisible);

  console.log("----", nodeId);

  return centerDialogVisible;
};
const closeCenterDialogVisible = (action) => {
  if (action == "delete") {
    centerDialogVisible.value = false;
    return;
  }
  centerDialogVisible.value = false;
  console.log("====-=-=-== ", centerDialogVisible, action);
  var alarmMes = {
    alarmId: -1,
    alarmList: "cluster.wise-nginx.nwise.www",
    alarmConfig: "dhagkjhduiaksuhdkujsabn\noiaudjkisajikjds\nihsdakhdijusah",
    dependence: "strong",
  };
  for (var i in alarmListGroup.value) {
    console.log("12311354xn,nm,h", alarmListGroup.value[i]);
    if (alarmListGroup.value[i].alarmId == alarmId) {
      alarmMes = alarmListGroup.value[i];
      break;
    }
  }

  console.log("-=0=-0=9213917838912 ", alarmMes);
  commitUrl = "http://10.9.228.2:8080/relayAlarm?RelayAlarmId=-1";

  axios
    .post(commitUrl, {
      action: action,
      data: {
        functionId: parseInt(nodeId),
        alertTernaryId: alarmMes.alarmId,
        alertTernary: alarmMes.alarmList,
        relayRelation: alarmMes.dependence,
      },
    })
    .then(function (response) {
      console.log("============ ", response);
    })
    .catch(function (error) {
      console.log(" =========== ", error);
    });

  return centerDialogVisible;
};

// {
//     "action":"mod",
//     "data":{
//         "relayAlarmId":1,
//         "alertTernary":"cluster.ala-root.www.all:finance_all_sla_alert",
//         "relayRelation":"strong",
//     }
// }
</script>

<style>
#RelySelect {
  width: 10px;
}
.infinite-list-wrapper {
  height: 300px;
  font-size: 10px;
  text-align: left;
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
