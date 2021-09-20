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
                "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI2NiIsIm5hbWUiOiJtLmZvY2FyZXRhQGFncmljb2x1cy5jb20iLCJyb2xlIjpbIkFkbWluaXN0cmF0b3IiLCJBcHByb3ZlcnMiLCJDYW5DaGFuZ2VGZWF0dXJlcyIsIkNhbkFkZEZlYXR1cmVzIl0sIm9yZ2FuaXphdGlvbl9pZCI6IjY1IiwidXNlcl9maXN0X25hbWUiOiJNaWNoZWxlIiwidXNlcl9sYXN0X25hbWUiOiJGb2NhcmV0YSIsImFncm9fb2YiOiIxODEiLCJ0b2tlbl91c2FnZSI6ImFjY2Vzc190b2tlbiIsImp0aSI6Ijg1MGNiMmJlLTE4ZTgtNGMwNy04MGY0LTBlMGUwN2NkMWE4NiIsInNjb3BlIjpbIm9wZW5pZCIsInByb2ZpbGUiLCJlbWFpbCJdLCJhenAiOiJhZ3JpY29sdXNfd2ViIiwibmJmIjoxNjMyMTIyMzU0LCJleHAiOjE2MzIxNjU1NTQsImlhdCI6MTYzMjEyMjM1NCwiaXNzIjoiaHR0cHM6Ly9zdGFnaW5nLWF1dGguYWdyaWNvbHVzLmNvbS8ifQ.5OzBtE1exZbR60r0sbo0zIr5NIS5M6YLZzVdQtBHKhw",
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
          this.formattedData[i] = ["AGRICOLUS", this.getDataResponse[i].name];
        }
      }
    },
    findNode(event) {
      for (let i = 0; i < this.getDataResponse.length; i++) {
        if (this.getDataResponse[i].name === event.point.name)
          var currentNode = this.getDataResponse[i];
      }
      this.updateGraph(currentNode);
    },
    updateGraph(node) {
      let index = 0;
      this.formattedData = [];
      for (let i = 0; i < this.getDataResponse.length; i++) {
        let level = node.level + 1;
        if (this.getDataResponse[i].level === level) {
          if (this.getDataResponse[i].parentId === node.id) {
            // console.log(this.getDataResponse[i]);
            this.formattedData[index] = [
              node.name,
              this.getDataResponse[i].name,
            ];
            index++;
            this.createGraph();
          }
        }
      }
      console.log(this.formattedData);
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
                id: "AGRICOLUS",
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
