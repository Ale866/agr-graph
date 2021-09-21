<template>
  <section>
    <div id="container"></div>
  </section>
</template>

<script lang="ts">
import Vue from "vue";
import axios from "axios";
import Highcharts from "highcharts";
import HighchartsNetworkgraph from "highcharts/modules/networkgraph";

export default Vue.extend({
  data() {
    return {
      getDataResponse: [],
      formattedData: [],
      mainNode: {
        name: "AGRICOLUS",
        id: 273,
        level: 0,
      },
      previousNode: "",
      storedData: [],
    };
  },
  methods: {
    async getData() {
      await axios
        .get(
          `https://staging-commerce-api.agricolus.com/api/organizations/273/tree?level=100`,
          {
            headers: {
              Authorization:
                "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI2NiIsIm5hbWUiOiJtLmZvY2FyZXRhQGFncmljb2x1cy5jb20iLCJyb2xlIjpbIkFkbWluaXN0cmF0b3IiLCJBcHByb3ZlcnMiLCJDYW5DaGFuZ2VGZWF0dXJlcyIsIkNhbkFkZEZlYXR1cmVzIl0sIm9yZ2FuaXphdGlvbl9pZCI6IjY1IiwidXNlcl9maXN0X25hbWUiOiJNaWNoZWxlIiwidXNlcl9sYXN0X25hbWUiOiJGb2NhcmV0YSIsImFncm9fb2YiOiIxODEiLCJ0b2tlbl91c2FnZSI6ImFjY2Vzc190b2tlbiIsImp0aSI6IjA2OGZiOTQyLWMzM2EtNGYzYy05YTEwLWY2OTRlNDUyODRkNCIsInNjb3BlIjpbIm9wZW5pZCIsInByb2ZpbGUiLCJlbWFpbCJdLCJhenAiOiJhZ3JpY29sdXNfd2ViIiwibmJmIjoxNjMyMjA5MTI2LCJleHAiOjE2MzIyNTIzMjYsImlhdCI6MTYzMjIwOTEyNiwiaXNzIjoiaHR0cHM6Ly9zdGFnaW5nLWF1dGguYWdyaWNvbHVzLmNvbS8ifQ.C7atX2PjQuFcvrDCEzja52kvq9ypsL5Pcnsq2_YdDm8",
            },
          }
        )
        .then((resp) => {
          this.getDataResponse = resp.data.filter(
            (f) => !(f.type == "Customer" && f.parentId == "273")
          );
        });
    },
    displayFirstLevel() {
      for (let i = 0; i < this.getDataResponse.length; i++) {
        if (this.getDataResponse[i].level === 1) {
          this.formattedData[i] = [
            this.mainNode.name,
            this.getDataResponse[i].name,
          ];
        }
      }
    },
    resetData() {
      this.previousNode = "";
      this.storedData = [];
    },
    findNode(event) {
      for (let i = 0; i < this.getDataResponse.length; i++) {
        if (this.getDataResponse[i].name === event.point.name)
          var currentNode = this.getDataResponse[i];
      }
      if (event.point.name === "AGRICOLUS") {
        this.resetData();
        currentNode = this.mainNode;
      }
      this.updateGraph(currentNode);
    },
    updateGraph(node) {
      let index = 1;
      this.formattedData = [];
      let level = node.level + 1;
      let flag = false;

      if (node.parentId === this.mainNode.id) {
        this.storedData[0] = [this.mainNode.name, node.name];
      }
      if (this.storedData.length > 0) {
        this.formattedData[0] = this.storedData[0];
      }

      for (let i = 0; i < this.getDataResponse.length; i++) {
        if (this.getDataResponse[i].level === level) {
          if (this.getDataResponse[i].parentId === node.id) {
            if (!(this.previousNode === node)) {
              if (this.previousNode != "" && flag === false) {
                this.formattedData[index] = [this.previousNode.name, node.name];
                index++;
              }
              this.formattedData[index] = [
                node.name,
                this.getDataResponse[i].name,
              ];
              index++;
              flag = true;
            }
          }
        }
      }
      console.log(this.formattedData);

      if (flag) {
        this.previousNode = node;
      }
      if (this.formattedData.length > 1) {
        this.createGraph();
      }
    },
    createGraph() {
      let body = document.getElementsByTagName("body")[0];
      HighchartsNetworkgraph(Highcharts);
      (Highcharts as any).chart("container", {
        chart: {
          type: "networkgraph",
          plotBorderWidth: 0,
        },
        title: {
          text: "",
        },
        plotOptions: {
          networkgraph: {
            layoutAlgorithm: {
              enableSimulation: true,
            },
            keys: ["from", "to"],
            events: {
              click: (event) => {
                this.findNode(event);
              },
            },
          },
        },
        series: [
          {
            marker: {
              radius: 15,
              lineWidth: 0,
            },
            dataLabels: {
              enabled: true,
              linkFormatter: function () {
                return "";
              },
              linkFormat: "",
            },
            layoutAlgorithm: {
              enableSimulation: true,
              friction: -0.9,
              maxIterations: 1,
              initialPositions: function () {
                var chart = this.series[0].chart,
                  width = chart.plotWidth,
                  height = chart.plotHeight;

                this.nodes.forEach(function (node, i) {
                  if (i === 0) {
                    node.plotX = body.clientWidth / 2;
                    node.plotY = 100;
                  } else if (i !== 0) {
                    node.plotX = Math.floor(
                      Math.random() * (body.clientWidth - 200 + 1) + 100
                    );
                    node.plotY = Math.ceil(Math.random() * 500 + 200);
                  }
                });
              },
            },
            nodes: [
              {
                id: this.mainNode.name,
                dataLabels: {
                  enabled: true,
                  verticalAlign: "bottom",
                  style: {
                    fontSize: "15pt",
                  },
                },
                marker: {
                  radius: 30,
                  fillColor: "green",
                },
              },
            ],
            id: "chart",
            data: this.formattedData,
          },
        ],
      });
    },
  },
  async mounted() {
    await this.getData();
    this.displayFirstLevel();
    this.createGraph();
  },
});
</script>

<style>
* {
  box-sizing: border-box;
}
body {
  margin: 0;
}
div {
  width: 100vw;
  height: 100vh;
  background-color: rgb(198, 223, 192);
}
path,
text {
  cursor: pointer;
}
</style>
