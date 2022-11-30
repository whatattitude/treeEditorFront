<template>
  <div id="box">
    <div>
      <div id="editor">
        <div id="node">
          <el-select v-model="value" placeholder="Select" class="nodeSelect">
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
          <AlarmConfigList></AlarmConfigList>
        </div>
        <div id="editor_item">
          <div id="addElem"></div>
          <el-radio-group v-model="radio" id="select">
            <el-radio :label="3">Option A</el-radio>
            <el-radio :label="6">Option B</el-radio>
            <el-radio :label="9">Option C</el-radio>
          </el-radio-group>
          <AlarmConfigList></AlarmConfigList>
        </div>
      </div>
    </div>
    <button @click="getNodeTree">刷新节点</button>
    <div id="container"></div>
  </div>
</template>

<style>
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
  padding: 10px;
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

export default {
  name: "Started",

  components: {
    AlarmConfigList,
  },
  data() {
    return {
      graph: "",
      value: ref(""),
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
        id: "A",
        description: "A",
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
            description: "A2",
            children: [
              {
                id: "A21",
                description: "A21",
                children: [
                  { id: "A211", description: "A211" },
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
          graph.on("node:click", (e) => {
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
    G6.registerNode("card-node", {
      draw: function drawShape(cfg, group) {
        const r = 2;
        const color = "#5B8FF9";
        const w = cfg.size[0];
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
        indent: 200,
        getHeight: () => {
          return 60;
        },
      },
    });
    this.graph = graph;
    graph.data(this.data);
    graph.render();
    graph.fitView();
    graph.on("node:click", (e) => {
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
  },
  updated() {},
};
</script>
