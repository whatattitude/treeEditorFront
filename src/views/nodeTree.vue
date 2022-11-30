<template>
  <div id="box">
    <div>
      <div id="editor">
        <div id="node">
          <el-select
            v-model="value"
            placeholder="nodeId"
            class="nodeSelect"
            :key="nodeSelectedId"
          >
            <el-option
              v-for="item in options"
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
              <el-button type="primary">Primary</el-button>
            </el-row>
          </div>
          <AlarmConfigList
            url="adsaa"
            nodeId="nodeSelectedId"
          ></AlarmConfigList>
        </div>
        <div id="editor_item">
          <div id="addElem"></div>
          <div id="fc">功能异常判定规则</div>
          <el-radio-group v-model="radio" id="select" @change="radioChange">
            <el-radio :label="1">仅有一个弱依赖规则异常时</el-radio>
            <el-radio :label="2">50%弱依赖规则异常率</el-radio>
            <el-radio :label="3">100%弱依赖规则异常率</el-radio>
          </el-radio-group>
          <AlarmConfigList
            url="qweqw"
            nodeId="nodeSelectedId"
          ></AlarmConfigList>
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
import { leftRotate } from "@antv/matrix-util/lib/ext";
import { objectToString } from "@vue/shared";

export default {
  name: "Started",

  components: {
    AlarmConfigList,
  },
  data() {
    return {
      nodeSelectedId: "A1",
      graph: "",
      value: "Select",
      options: [
        {
          value: "Option1",
          label: "Option1",
        },
        {
          value: "Option2",
          label: "Option2",
          disabled: true,
        },
        {
          value: "Option3",
          label: "Option3",
        },
        {
          value: "Option4",
          label: "Option4",
        },
        {
          value: "Option5",
          label: "Option5",
        },
      ],
      radio: ref(3),
      input: "a",
      node_title: "A",
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
            id: "A2",
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
                  { id: "A212", description: "A212" },
                ],
              },
              {
                id: "A22",
                description: "A22",
              },
            ],
          },
        ],
      },
      count: 1,
    };
  },
  methods: {
    radioChange: function () {
      console.log("==== ", this.radio);
    },
    listJson: function (jsonList, idList) {
      for (var key in jsonList) {
        console.log("====", typeof jsonList[key], jsonList[key]);
        if (Array.isArray(jsonList[key]) && key == "children") {
          for (var i in jsonList[key]) {
            console.log(Array.isArray(i), jsonList[key][i], key, " 000");
            this.listJson(jsonList[key][i], idList);
          }
        }
        if (key == "id") {
          console.log(typeof idList);
          idList.push({ value: jsonList[key], label: jsonList[key] });
        }
      }
      return idList;
    },
    SelectIdFromOptions: function (nodeSelectedId) {
      for (var i in this.options) {
        if (this.options[i].value == nodeSelectedId) {
          this.options[i].disabled = true;
        } else {
          delete this.options[i].disabled;
        }
      }
      console.log("111111 ", this.options);
    },
    CreateSelectList: function (nodeSelectedId) {
      var selectList = [{}];
      const idList = [];
      this.listJson(this.data, idList);
      console.log("===", idList);
      this.options = idList;
      this.SelectIdFromOptions(nodeSelectedId);
    },
    getNodeTree: function (name) {
      var that = this;
      if (name == "") {
        name = "default";
      }

      axios
        .get("http://127.0.0.1:4523/m1/1998892-0-default/nodeTree", {
          params: {
            name: name,
          },
        })
        .then(function (response) {
          that.data = response.data;

          that.graph.changeData(that.data);

          that.graph.fitView();
          that.graph.on("node:click", (e) => {
            if (e.target.get("name") === "collapse-icon") {
              e.item.getModel().collapsed = !e.item.getModel().collapsed;
              graph.setItemState(
                e.item,
                "collapsed",
                e.item.getModel().collapsed
              );
              graph.layout();
            }
          });

          if (typeof window !== "undefined")
            window.onresize = () => {
              if (!graph || graph.get("destroyed")) return;
              if (
                !container ||
                !container.scrollWidth ||
                !container.scrollHeight
              )
                return;
              graph.changeSize(container.scrollWidth, container.scrollHeight);
            };
          console.log(response.data);
        })
        .catch(function (error) {
          console.log(error);
        });

      console.log("====", that.data);

      return;
    },
  },
  mounted() {
    var nodeHeight = 100;
    var nodeWith = 60;
    G6.registerNode("card-node", {
      draw: function drawShape(cfg, group) {
        console.log("2123", cfg);
        const r = 2;
        const color = "#5B8FF9";
        cfg.size[0] = cfg.size[0] + nodeWith;
        const w = cfg.size[0];
        cfg.size[1] = cfg.size[1] + nodeHeight;
        const h = cfg.size[1];
        const shape = group.addShape("rect", {
          attrs: {
            x: -w / 2,
            y: -h / 2,
            width: w, //200,
            height: h, // 60
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
            height: h / 2 - nodeHeight / 2, // 60
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
        default: ["drag-canvas"],
      },
      defaultNode: {
        type: "card-node",
        size: [100, 40],
      },
      defaultEdge: {
        type: "cubic-horizontal",
        style: {
          endArrow: true,
        },
      },
      layout: {
        type: "indented",
        direction: "LR",
        dropCap: false,
        indent: nodeWith + 100 + 200,
        getHeight: () => {
          return nodeHeight + 40;
        },
      },
    });
    this.graph = graph;
    graph.data(this.data);
    graph.render();
    graph.fitView();
    graph.on("node:click", (e) => {
      console.log("i am clicked");
      this.CreateSelectList(e.item._cfg.id);
      console.log("---", this.nodeSelectedId);
      this.value = e.item._cfg.id;
      console.log("---", this.nodeSelectedId);

      if (e.target.get("name") === "collapse-icon") {
        e.item.getModel().collapsed = !e.item.getModel().collapsed;
        graph.setItemState(e.item, "collapsed", e.item.getModel().collapsed);
        graph.layout();
      }
    });

    if (typeof window !== "undefined")
      window.onresize = () => {
        if (!graph || graph.get("destroyed")) return;
        if (!container || !container.scrollWidth || !container.scrollHeight)
          return;
        graph.changeSize(container.scrollWidth, container.scrollHeight);
      };
    this.CreateSelectList();
  },
  updated() {},
};
</script>
