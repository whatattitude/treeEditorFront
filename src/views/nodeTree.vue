<template>
  <div id="box">
    <div>
      <div id="editor">
        <div id="node">
          <el-select
            v-model="rootValue"
            placeholder="nodeId1"
            class="nodeSelect"
            @change="rootSelect"
            :key="selectount"
          >
            <el-option
              v-for="item in rootNode"
              :key="item.value"
              :label="item.label"
              :value="item.value"
              :disabled="item.disabled"
          /></el-select>
          <el-select
            v-model="leafValue"
            placeholder="nodeId2"
            class="nodeSelect"
            @change="leafSelect"
            :key="selectount"
          >
            <el-option
              v-for="item in leafNode"
              :key="item.value"
              :label="item.label"
              :value="item.value"
              :disabled="item.disabled"
          /></el-select>
        </div>

        <div id="editor_item">
          <div id="addElem">
            <el-input v-model="input" placeholder="Please input" clearable />
            <el-row class="mb-4">
              <el-button type="primary" @click="getAlarmList">搜索</el-button>
            </el-row>
          </div>
          <AllAlarmList ref="DataReflush1"></AllAlarmList>
        </div>
        <div id="editor_item">
          <div id="addElem"></div>
          <div id="fc">功能异常判定规则</div>
          <el-radio-group
            v-model="ratio"
            :key="funcCount"
            id="select"
            @change="ratioChange"
          >
            <el-radio :label="1">仅有一个弱依赖规则异常时</el-radio>
            <el-radio :label="2">50%弱依赖规则异常率</el-radio>
            <el-radio :label="3">100%弱依赖规则异常率</el-radio>
          </el-radio-group>
          <div id="fc">功能异常报警可信度</div>
          <el-radio-group
            v-model="reliability"
            id="select"
            :key="funcCount"
            @change="ratioChange"
          >
            <el-radio :label="1">不可信</el-radio>
            <el-radio :label="2">可信</el-radio>
            <el-radio :label="3">多误报</el-radio>
          </el-radio-group>

          <AlarmConfigList ref="DataReflush2"></AlarmConfigList>
        </div>
      </div>
    </div>
    <button @click="getNodeTree">刷新节点</button>
    <div id="container"></div>
  </div>
</template>

<style>
#fc {
  text-align: center;
}
.nodeSelect {
  min-width: 100%;
  flex: 1;
}
#node {
  width: 100%;
  display: flex;
  flex-direction: column;
}
#select {
  display: flex;
  margin-bottom: 10px;
  margin-left: 10px;
}
#editor_item {
  border: 2px solid rgb(113, 95, 95);
  margin-top: 10px;
}
#dataList {
  display: flex;
  flex-direction: column;
}
#box {
  border: 10px solid skyblue;
  background-color: darkcyan;
}
#editor {
  border: 1px solid yellow;
  background: grey;
  position: absolute;
  right: 0;
  height: 100vh;
  width: 30vw;
  padding: 10px;
}
#container {
  height: 100vh;
  width: 100vw;
}
#addElem {
  display: flex;
  margin: 10px;
}
</style>

<script>
import G6 from "@antv/g6";
import { ref } from "vue";
import axios from "axios";

import AlarmConfigList from "../components/icons/AlarmConfigList.vue";
import AllAlarmList from "../components/icons/AllAlarmList.vue";
import { leftRotate } from "@antv/matrix-util/lib/ext";

export default {
  name: "Started",

  components: {
    AlarmConfigList,
    AllAlarmList,
  },
  data() {
    return {
      nodeId: ref("1"),
      selectount: 0,
      funcCount: ref(0),
      graph: "",
      rootValue: " 分类选择（根节点）",
      leafValue: "功能选择（叶子节点）",
      options: [
        {
          value: "Option2",
          label: "Option2",
          disabled: true,
        },
      ],
      ratio: ref(3),
      reliability: ref(3),
      input: ref("输入报警三元组搜索"),
      node_title: "A",
      dataBack: {},
      data: {
        id: "爱很简单看见啊汇顶科技爱的海洋",
        description: "A",
        ada: { id: "A1", description: "A1" },
        children: [
          {
            id: "A1",
            description: "A1",
            children: [
              { id: "A11", description: "A11" },
              { id: "A12", description: "A12" },
              { id: "A13", description: "A13" },
              { id: "A14", description: "A14" },
            ],
          },
          {
            id: "1:A2",
            description:
              "cluster.wise-nginx-home.nwise.hbe:service:结果页流量一次\nlaidujsalidjoasjdilkasjdsilakujdsalkdjaskdjhas",
            children: [
              {
                id: "A21",
                description: "A21",
                children: [
                  {
                    id: "A211",
                    description:
                      "cluster.wise-nginx-home.nwise.hbe:service:结果页流量一次\naidujsalidjoasjdilkasjdsilak\naidujsalidjoasjdilkasjdsilak\naidujsalidjoasjdilkasjdsilak",
                  },
                  { id: "1:A212", description: "A212" },
                ],
              },
              {
                id: "2:A22",
                description: "A22",
              },
            ],
          },
        ],
      },
      rootNode: [],
      leafNode: [],
      count: 1,
      selectListFuncData: [],
      nodeSelectedId: "",
    };
  },
  methods: {
    funcConfig(nodeId) {
      console.log(
        "12313132132131232142314112312       =====   ",
        this.ratio,
        this.reliability
      );
      var that = this;

      axios
        .get(
          "http://10.9.228.2:8080/proPatternAndFunctionPiont?nodeId=" + nodeId
        )
        .then(function (response) {
          console.log("-------------------------------------");
          that.ratio = response.data.Data[0].ratio;
          var paramReliability = 0;
          switch (response.data.Data[0].reliability) {
            case "unreliable":
              paramReliability = 1;
              break; //停止执行，跳出switch
            case "reliable":
              paramReliability = 2;
              break; //停止执行，跳出switch
            case "half":
              paramReliability = 3;
              break; //停止执行，跳出switch
            default: //上述条件都不满足时，默认执行的代码
              paramReliability = 1;
          }
          that.reliability = paramReliability;
          funcCount++;
          console.log(
            "12313132132131232142314112312       =====   ",
            this.ratio,
            this.reliability
          );
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    getAlarmList() {
      var reqUrl =
        "http://10.9.228.2:8080/NoahAlarmList?alertTernary=" + this.input;
      this.$refs.DataReflush1.getDataAllAlarm(reqUrl);
    },
    nodeClick: function (e) {},
    DataReflush: function (url, nodeId) {
      console.log(this.leafNode);
      var hasNodeId = false;
      for (var i in this.leafNode) {
        if (this.leafNode[i].value == nodeId) {
          hasNodeId = true;
        }
      }
      console.log("=-132232===", url, nodeId, hasNodeId);
      if (hasNodeId) {
        var id = nodeId.split(":")[0];
        console.log("242133254das5dasd", url, id);
        this.funcConfig(id);
        this.$refs.DataReflush1.setParams(id);
        this.$refs.DataReflush2.getData(url, id);
        console.log(
          "242133254das5dasd===============================================================================",
          this.$refs,
          url,
          id
        );
      } else {
        return;
      }
    },
    rootSelect: function (value) {
      console.log("======0896tvgmnjk ", this.data);

      if (value == this.dataBack.id) {
        this.graph.changeData(this.data);
        this.graph.render();
        this.graph.fitView();

        this.rootNode = [];
        this.leafNode = [];
        this.listJson(this.data);
        return;
      }

      var childrenList = this.dataBack.children;

      while (Array.isArray(childrenList)) {
        if (childrenList.length == 0) {
          childrenList = this.selectListFuncData.pop().children;
          continue;
        }
        console.log(
          "=-=-=-=-",
          value,
          Array.isArray(childrenList),
          childrenList.length,
          childrenList
        );
        for (var i in childrenList) {
          if (childrenList[i].id == value) {
            console.log("21346546==== ", childrenList[i], value);

            this.graph.changeData(childrenList[i]);

            this.graph.render();
            this.graph.fitView();

            console.log("=============65746834 ", this.data);

            var rootNodeBack = this.rootNode;
            this.rootNode = [];
            this.leafNode = [];
            this.listJson(childrenList[i]);
            this.rootNode = rootNodeBack;
            return;
          }
          console.log("压", childrenList[i]);
          this.selectListFuncData.push(childrenList[i]);
        }
        childrenList = this.selectListFuncData.pop().children;
        console.log("=-=-2415465=-=", childrenList);
        console.log(
          "=-=-=-=-",
          value,
          Array.isArray(childrenList),
          childrenList.length,
          childrenList
        );
      }
    },
    leafSelect: function (value) {
      console.log("=-=-=-=-", value);
      this.DataReflush("http://10.9.228.2:8080/alarmList", value);

      //this.CreateSelectList();
    },

    ratioChange: function () {
      var action = "UPDATE";

      var id = "";
      for (var i in this.leafNode) {
        console.log(this.leafNode);
        if (this.leafNode[i].disabled == true) {
          id = parseInt(this.leafNode[i].value.split(":")[0]);
        }
      }
      console.log("======== ", this.ratio, id);
      var commitUrl = "http://10.9.228.2:8080/relayRelation?id=" + id;
      var paramReliability = "unreliable";

      switch (this.reliability) {
        case 1:
          paramReliability = "unreliable";
          break; //停止执行，跳出switch
        case 2:
          paramReliability = "reliable";
          break; //停止执行，跳出switch
        case 3:
          paramReliability = "half";
          break; //停止执行，跳出switch
        default: //上述条件都不满足时，默认执行的代码
          paramReliability = "reliable";
      }

      axios
        .post(commitUrl, {
          action: action,
          data: {
            id: id,
            ratio: this.ratio,
            reliability: paramReliability,
          },
        })
        .then(function (response) {
          console.log("============ ", response);
        })
        .catch(function (error) {
          console.log(" =========== ", error);
        });
    },

    listJson: function (jsonList) {
      var hasChildren = false;
      for (var key in jsonList) {
        console.log("====", typeof jsonList[key], jsonList[key]);
        if (
          Array.isArray(jsonList[key]) &&
          key == "children" &&
          jsonList[key].length != 0
        ) {
          hasChildren = true;
          console.log(
            Array.isArray(jsonList[key]),
            jsonList[key],
            key,
            jsonList[key].length,
            " 111111111111000"
          );

          for (var i in jsonList[key]) {
            this.listJson(jsonList[key][i]);
          }
        }
      }
      if (!Array.isArray(jsonList[key])) {
        if (hasChildren == true) {
          this.rootNode.push({ value: jsonList.id, label: jsonList.id });
        } else {
          this.leafNode.push({ value: jsonList.id, label: jsonList.id });
        }
      }
    },
    SelectIdFromOptions: function (nodeSelectedId) {
      for (var i in this.leafNode) {
        if (this.leafNode[i].value == nodeSelectedId) {
          this.leafNode[i].disabled = true;
          this.leafValue = nodeSelectedId;
          this.rootValue = "当前选中为叶子节点";
        } else {
          delete this.leafNode[i].disabled;
        }
      }
      for (var i in this.rootNode) {
        if (this.rootNode[i].value == nodeSelectedId) {
          this.rootNode[i].disabled = true;
          this.rootValue = nodeSelectedId;
          this.leafValue = "当前选中为根节点";
        } else {
          delete this.rootNode[i].disabled;
        }
      }
    },
    CreateSelectList: function (nodeSelectedId) {
      //var selectList = [{}];
      //const idList = [];

      //this.listJson(this.data);
      //console.log("===", idList);
      //this.options = idList;
      this.SelectIdFromOptions(nodeSelectedId);
      this.selectount++;

      console.log(nodeSelectedId, this.selectount);
    },
    getNodeTree: function (name) {
      var that = this;
      if (name == "") {
        name = "default";
      }

      axios
        .get("http://10.9.228.2:8080/nodeTree", {
          params: {},
        })
        .then(function (response) {
          that.data = response.data.Data;

          that.graph.changeData(that.data);
          that.graph.render();
          that.graph.fitView();

          console.log("adasdsalkdjkl ====== ", that.data);
          that.dataBack = that.data;
          that.rootNode = [];
          that.leafNode = [];
          that.listJson(that.data);
        })
        .catch(function (error) {
          console.log(error);
        });

      console.log("====", that.data);

      return;
    },
  },
  mounted() {
    this.dataBack = this.data;

    var withNode = 0;
    var heightNode = 40;
    G6.registerNode("card-node", {
      draw: function drawShape(cfg, group) {
        const r = 2;
        const color = "#5B8FF9";
        cfg.size[0] = cfg.size[0] + withNode;
        const w = cfg.size[0];
        cfg.size[1] = cfg.size[1];
        const h = cfg.size[1];
        const shape = group.addShape("rect", {
          attrs: {
            x: -w / 2,
            y: -h / 2,
            width: w, //200,
            height: h + heightNode, // 60
            stroke: color,
            radius: r,
            fill: "#fff",
          },
          name: "main-box",
          draggable: true,
        });

        group.addShape("rect", {
          attrs: {
            x: -w / 2,
            y: -h / 2,
            width: w, //200,
            height: h / 2, // 60
            fill: color,
            radius: [r, r, 0, 0],
          },
          name: "title-box",
          draggable: true,
        });

        // title text
        group.addShape("text", {
          attrs: {
            textBaseline: "top",
            x: -w / 2 + 8,
            y: -h / 2 + 2,
            lineHeight: 20,
            fontSize: 14,

            text: cfg.id,
            fill: "#fff",
          },
          name: "title",
        });

        cfg.children &&
          group.addShape("marker", {
            attrs: {
              x: w / 2,
              y: 0,
              r: 6,
              cursor: "pointer",
              symbol: cfg.collapsed ? G6.Marker.expand : G6.Marker.collapse,
              stroke: "#666",
              lineWidth: 1,
              fill: "#fff",
            },
            name: "collapse-icon",
          });
        group.addShape("text", {
          attrs: {
            textBaseline: "top",
            textAlign: "left",

            x: -w / 2 + 8,
            y: -h / 2 + 24,
            lineHeight: 20,
            text: cfg.description,
            fill: "rgba(0,0,0, 1)",
          },
          name: `description`,
        });
        return shape;
      },
      setState(name, value, item) {
        if (name === "collapsed") {
          const marker = item
            .get("group")
            .find((ele) => ele.get("name") === "collapse-icon");
          const icon = value ? G6.Marker.expand : G6.Marker.collapse;
          marker.attr("symbol", icon);
        }
      },
    });

    var container = document.getElementById("container");
    const width = container.scrollWidth;
    const height = container.scrollHeight || 500;

    const graph = new G6.TreeGraph({
      container: "container",
      width,
      height,
      modes: {
        default: ["drag-canvas", "zoom-canvas"],
      },
      defaultNode: {
        type: "card-node",
        size: [200, 40],
      },
      defaultEdge: {
        type: "cubic-horizontal",
        style: {
          endArrow: true,
        },
      },
      layout: {
        type: "indented",
        direction: "H",
        dropCap: true,
        indent: 200,
        getHeight: () => {
          return 100;
        },
      },
    });
    this.graph = graph;
    graph.data(this.data);
    graph.render();
    graph.fitView();
    graph.on("node:click", (e) => {
      console.log("i am clicked");
      console.log("------------  ", e);
      this.CreateSelectList(e.item._cfg.id);
      //console.log("--132-", this.nodeSelectedId);.3
      this.nodeSelectedId = e.item._cfg.id;
      //this.value = e.item._cfg.id;
      console.log("-1321--", this.nodeSelectedId);
      this.DataReflush("http://10.9.228.2:8080/alarmList", e.item._cfg.id);

      if (e.target.get("name") === "collapse-icon") {
        e.item.getModel().collapsed = !e.item.getModel().collapsed;
        graph.setItemState(e.item, "collapsed", e.item.getModel().collapsed);
        graph.layout();
      }
      console.log("___ this.leafNode ", this.leafNode);
      console.log("this.rootNode ", this.rootNode);
    });

    if (typeof window !== "undefined")
      window.onresize = () => {
        if (!graph || graph.get("destroyed")) return;
        if (!container || !container.scrollWidth || !container.scrollHeight)
          return;
        graph.changeSize(container.scrollWidth, container.scrollHeight);
      };

    console.log("adshgajbhv ", this.data);
    //this.CreateSelectList();
    this.rootNode = [];
    this.leafNode = [];
    this.listJson(this.data);
    console.log("adshg541654 ", this.data);
  },
  updated() {},
};
</script>
