<template>
  <section>
    <div id="container"></div>
  </section>
</template>

<script lang="ts">
import Vue from "vue";
import axios from "axios";
import Highcharts, { numberFormat } from "highcharts";
import HighchartsNetworkgraph from "highcharts/modules/networkgraph";

export default Vue.extend({
  data() {
    return {
      getDataResponse: [],
      formattedData: [],
      slicedData: [],
      totalElements: 10,
      mainNode: {
        name: "AGRICOLUS",
        email: "AGRICOLUS",
        id: 273,
        level: 0,
      },
      previousNode: "",
      storedData: [],
      currentPage: 0,
      prova: true,
      btn: false,
      temp: Object,
      displayNext: true,
      displayPrev: true,
      firstLevelArray: [],
      storePath: [],
      array: [],
      array2: [],
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
                "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI2NiIsIm5hbWUiOiJtLmZvY2FyZXRhQGFncmljb2x1cy5jb20iLCJyb2xlIjpbIkFkbWluaXN0cmF0b3IiLCJBcHByb3ZlcnMiLCJDYW5DaGFuZ2VGZWF0dXJlcyIsIkNhbkFkZEZlYXR1cmVzIl0sIm9yZ2FuaXphdGlvbl9pZCI6IjY1IiwidXNlcl9maXN0X25hbWUiOiJNaWNoZWxlIiwidXNlcl9sYXN0X25hbWUiOiJGb2NhcmV0YSIsImFncm9fb2YiOiIxODEiLCJ0b2tlbl91c2FnZSI6ImFjY2Vzc190b2tlbiIsImp0aSI6IjZjMDVmZjM5LTJlNzYtNGE4NC04NTBhLTAwNmM1ODZmNTA4YyIsInNjb3BlIjpbIm9wZW5pZCIsInByb2ZpbGUiLCJlbWFpbCJdLCJhenAiOiJhZ3JpY29sdXNfd2ViIiwibmJmIjoxNjMzMjc1MjY2LCJleHAiOjE2MzMzMTg0NjYsImlhdCI6MTYzMzI3NTI2NiwiaXNzIjoiaHR0cHM6Ly9zdGFnaW5nLWF1dGguYWdyaWNvbHVzLmNvbS8ifQ.NuI_obI0Iiy4OHv5CYS0618Bx6sBVDNpGRz794svTjU",
            },
          }
        )
        .then((resp) => {
          this.getDataResponse = resp.data.filter(
            (f) => !(f.type == "Customer" && f.parentId == "273")
          );
          this.hasChild();
          for (let i = 0; i < this.getDataResponse.length; i++) {
            // if(this.getDataResponse[i].email == "michele.focareta+150@gmail.com"){
            //   this.getDataResponse[i].email = "michele.focareta+150@gmail.com" + i
            // }
            // console.log(this.getDataResponse[i]);
          }
        });
    },
    hasChild() {
      for (let i = 0; i < this.getDataResponse.length; i++) {
        for (let j = 0; j < this.getDataResponse.length; j++) {
          if (this.getDataResponse[i].id == this.getDataResponse[j].parentId) {
            this.getDataResponse[i].hasChild = true;
          }
        }
      }
    },
    displayFirstLevel(n?: number) {
      for (let i = 0; i < this.getDataResponse.length; i++) {
        if (this.getDataResponse[i].level === 1) {
          this.formattedData[i] = [
            this.mainNode.id,
            this.getDataResponse[i].id,
            this.getDataResponse[i].name,
          ];
        }
      }
      this.slicedData = [];
      this.formattedData = this.formattedData.sort((a, b) =>
        a[2].toUpperCase().localeCompare(b[2].toUpperCase())
      );

      let e = this.totalElements;
      this.currentPage = this.currentPage + (n == undefined ? 0 : n);
      this.currentPage =
        this.currentPage < 0
          ? 0
          : this.currentPage > Math.floor(this.formattedData.length / e)
          ? this.currentPage - 1
          : this.currentPage;

      let page = this.currentPage;
      this.slicedData = this.formattedData.slice(
        e * page,
        page == 0 ? this.totalElements : e * page * 2
      );
    },
    findNode(event) {
      for (let i = 0; i < this.getDataResponse.length; i++) {
        if (this.getDataResponse[i].id === event.point.name)
          var currentNode = this.getDataResponse[i];
      }
      if (event.point.name === 273) {
        this.resetData();
        currentNode = this.mainNode;
      }
      this.updateGraph(currentNode);
    },
    resetData() {
      this.previousNode = "";
      this.storedData = [];
      this.currentPage = 0;
    },
    updateGraph(node) {
      this.slicedData = [];
      let level = node.level + 1;
      let flag = false;
      let prevNode = this.previousNode;
      let diff = prevNode.level - node.level;

      if (
        node.level <= prevNode.level ||
        node.level == 0 ||
        prevNode.level == 0
      ) {
        this.storePath = [];
      }
      this.slicedData = this.formattedData.slice(
        this.totalElements * this.currentPage,
        this.currentPage == 0
          ? this.totalElements
          : this.totalElements * this.currentPage * 2
      );

      // if (node.id == 273) {

      //   this.slicedData = this.slicedData.sort((a, b) =>
      //     a[1].toUpperCase().localeCompare(b[1].toUpperCase())
      //   );
      //   this.slicedData = this.slicedData.slice(0, this.totalElements);
      //   console.log(this.totalElements);

      // }

      // if (node.parentId == this.mainNode.id) {
      //   this.storedData[0] = [this.mainNode.name, node.email];
      // }

      // if (this.storedData.length) {
      //   this.slicedData[0] = this.storedData[0];
      // }

      let response = this.getDataResponse.filter(
        (e) => e.level == level && e.parentId == node.id
      );

      if (response.length) {
        if (node != this.mainNode) {
          this.prova = false;
          this.displayNext = true;
          this.displayPrev = true;
        } else {
          this.prova = true;
          this.displayNext = true;
          this.displayPrev = true;
        }

        // if (prevNode.id == node.parentId) {
        //   this.slicedData.push([prevNode.email, node.email]);
        // }

        // if (diff >= 1) {
        //   while (diff != 0) {
        //     this.storedData.pop();
        //     diff--;
        //   }
        // }

        // for (let i = 1; i <= this.storedData.length; i++) {
        //   this.slicedData.push(this.storedData[i]);
        // }
      }
      this.temp = node;

      for (let i = 0; i < response.length; i++) {
        let r = response[i];
        this.storePath.push([node.id, r.id]);
        flag = true;
      }

      if (node.id != 273) {
        for (let i in this.storePath) {
          this.slicedData.push(this.storePath[i]);
        }
      }

      let arrPath = [];
      if (level > 2) {
        var cycleNode = node;
        while (cycleNode.parentId != 273) {
          var a = this.getDataResponse.find((e) => e.id == cycleNode.parentId);
          arrPath.push([a.id, cycleNode.id]);
          cycleNode = a;
          // console.log(cycleNode.name);
        }
        var s = this.getDataResponse.filter((e) => e.parentId == cycleNode.id);
      }
      for (let i in s) {
        arrPath.push([cycleNode.id, s[i].id]);
      }
      // console.log(s);

      // console.log(arrPath);
      for (let i in arrPath) {
        this.slicedData.push(arrPath[i]);
      }

      if (node.level == prevNode.level && node.level > 1) {
        let lvl = 1;
        var pNode = this.getDataResponse.find((e) => e.id == node.parentId);
        var pNodeLink = [pNode.id, node.id];
        // console.log(pNode);

        var sibilings = this.getDataResponse.filter(
          (e) => e.parentId == node.parentId
        );

        sibilings.sort((a, b) => a.name - b.name);

        this.slicedData.push(pNodeLink);

        for (let i in sibilings) {
          this.slicedData.push([pNode.id, sibilings[i].id]);
          // console.log(this.slicedData);
        }
      }
      if (node.id == 273) {
        if (prevNode.id == 273) {
          this.slicedData = this.slicedData.sort((a, b) =>
            a[1].toUpperCase().localeCompare(b[1].toUpperCase())
          );
        }
        this.slicedData = this.slicedData.slice(0, this.totalElements);
      }
      // console.log(this.slicedData);

      if (flag) {
        this.previousNode = node;
      }

      if (this.slicedData.length > 1) {
        this.createGraph();
      }
    },
    displayNode(node, i, fNode) {
      let body = document.getElementsByTagName("body")[0];
      if (i == 0) {
        node.plotX = body.clientWidth / 2;
        node.plotY = 70;
      } else if (i != 0) {
        var t = this.getDataResponse.find((e) => e.id == node.id);
        // ! t è l'oggetto
        if (t.level == 1) {
          node.plotX =
            ((body.clientWidth - 200) / (this.totalElements - 1)) * (i - 1) +
            100;
          node.plotY = 250;
        }

        if (fNode != null) {
          var n = this.getDataResponse.find((e) => e.id == fNode.id);
          // ! n è l'oggetto padre
        }

        // ! fNode è l'istanza del nodo padre

        // ! i valori di this.array vengono cambiati perchè son messi dentro per reference

        if (t.level >= 2) {
          node.plotX = this.array[t.level- 2].plotX + (i - 10) * 100;
          node.plotY = t.level * 100 + 250;
        }

        node.color =
          t.type == "BusinessUnit"
            ? t.hasChild
              ? "#362dcc"
              : "#6788AF"
            : "yellow";
      }
    },
    findNames(key) {
      let ref;
      this.getDataResponse.forEach((el) => {
        if (el.id === key) {
          ref = el.name;
          return ref;
        }
        return ref;
      });
      return ref;
    },
    createGraph() {
      let body = document.getElementsByTagName("body")[0];
      // eslint-disable-next-line @typescript-eslint/no-this-alias
      let that = this;
      HighchartsNetworkgraph(Highcharts);
      (Highcharts as any).chart(
        "container",
        {
          chart: {
            type: "networkgraph",
            renderTo: "container",
            plotBorderWidth: 0,
          },
          title: {
            text: "Agricolus Graph",
          },
          tooltip: {
            formatter: function() {
              let dummyArray = [];
              dummyArray = that.getDataResponse;
              dummyArray.unshift(that.mainNode);
              const t = dummyArray.find((e) => e.id == this.key);

              let countBU = 0,
                countC = 0;

              for (let i in dummyArray) {
                if (
                  t.id == dummyArray[i].parentId &&
                  dummyArray[i].type == "BusinessUnit"
                ) {
                  countBU++;
                } else if (
                  t.id == dummyArray[i].parentId &&
                  dummyArray[i].type == "Customer"
                ) {
                  countC++;
                }
              }

              let string;
              if (t.email != "AGRICOLUS") {
                // string = `<strong>Name:</strong> ${t.name} <br> <strong>Email:</strong> ${t.email} <br> <strong>id:</strong> ${t.id} <br> <strong>parentId:</strong> ${t.parentId}`;
                string = `<br> <strong>Name:</strong> ${t.name} <br> <strong>Email:</strong> ${t.email} <br> <strong>Number BU Child:</strong> ${countBU} <br><strong>Number Customer Child:</strong> ${countC} <br><strong>Id</strong> ${t.id}`;
              } else {
                string = `<strong>name:</strong> ${t.name} <br> <strong>id:</strong> ${t.id}<br> <strong>Number BU Child:</strong> ${countBU}`;
              }
              dummyArray.shift();
              return string;
            },
          },
          plotOptions: {
            networkgraph: {
              layoutAlgorithm: {
                enableSimulation: true,
              },
              keys: ["from", "to", "name"],
              events: {
                click: (event) => {
                  this.findNode(event);
                },
              },
            },
          },
          series: [
            {
              cursor: "pointer",
              marker: {
                symbol: "circle",
                radius: 15,
              },
              dataLabels: {
                enabled: true,
                linkFormat: "",
                formatter: function() {
                  let result = that.findNames(this.key) || "AGRICOLUS";
                  return result;
                },
              },
              layoutAlgorithm: {
                enableSimulation: true,
                friction: -0.9,
                maxIterations: 1,
                initialPositions: function() {
                  // var chart = this.series[0].chart
                  // width = chart.plotWidth,
                  // height = chart.plotHeight;
                  var a = [];
                  for (let i in this.nodes) {
                    a.push(
                      that.getDataResponse.find((e) => e.id == this.nodes[i].id)
                    );
                  }
                  a.shift();

                  for (let i in a) {
                    for (let j in a) {
                      if (a[i].id == a[j].parentId) {
                        var el = a[i];
                      }
                    }
                  }

                  var fatherNode;
                  if (el) {
                    fatherNode = this.nodes.find((e) => e.id == el.id);
                    that.array.push(fatherNode);
                  } else {
                    fatherNode = null;
                  }

                  console.log(that.array);

                  for (let i in this.nodes) {
                    that.displayNode(this.nodes[i], i, fatherNode);
                  }
                },
              },
              nodes: [
                {
                  id: this.mainNode.id,
                  dataLabels: {
                    enabled: true,
                    verticalAlign: "bottom",
                    style: {
                      fontSize: "15pt",
                    },
                  },
                  marker: {
                    symbol: "circle",
                    radius: 30,
                    fillColor: "#1d5644",
                  },
                },
              ],
              id: "chart",
              data: this.slicedData,
            },
          ],
        },
        function(chart): void {
          var point2 = chart.series[0],
            labelText = `Page ${that.currentPage + 1} of ${Math.ceil(
              that.formattedData.length / that.totalElements
            )}`;
          chart.renderer
            .text(labelText, 35, 59)
            .attr({
              zIndex: 5,
            })
            .css({
              fontSize: "18px",
            })
            .add();
          chart.renderer
            .rect(30, 30, 110, 45, 5)
            .attr({
              "stroke-width": 2,
              stroke: "black",
              fill: "white",
              zIndex: 4,
            })
            .add();

          if (
            that.displayNext &&
            that.currentPage <=
              Math.floor(that.formattedData.length / that.totalElements) - 1
          ) {
            that.renderNext(chart);
          }

          if (that.displayPrev && that.currentPage != 0) {
            that.renderPrevious(chart);
          }
        }
      );
    },
    renderPrevious(chart) {
      let body = document.getElementsByTagName("body")[0];

      var point = chart.series[0],
        text = chart.renderer
          .text(
            "Prev",
            (point.plotX = body.clientWidth - (body.clientWidth - 30)),
            (point.plotY = 305)
          )
          .attr({
            zIndex: 5,
          })
          .on("click", () => {
            if (this.displayPrev) {
              this.currentPage == 0
                ? (this.displayPrev = false)
                : (this.displayPrev = true);
              if (this.displayPrev) {
                this.displayFirstLevel(-1);
                this.createGraph();
              }
            }
          })
          .add(),
        box = text.getBBox();

      chart.renderer
        .rect(box.x - 5, box.y - 5, box.width + 10, box.height + 10, 5)
        .attr({
          fill: this.displayPrev
            ? this.currentPage == 0
              ? "gray"
              : "#1d5644"
            : "gray",
          stroke: "#1d5644",
          "stroke-width": 1,
          zIndex: 4,
        })
        .add();
    },
    renderNext(chart) {
      let body = document.getElementsByTagName("body")[0];
      var point = chart.series[0],
        text = chart.renderer
          .text(
            "Next",
            (point.plotX = body.clientWidth - 60),
            (point.plotY = 305)
          )
          .attr({
            zIndex: 5,
          })
          .on("click", () => {
            if (this.displayNext) {
              this.currentPage >
              Math.floor(this.formattedData.length / this.totalElements) - 1
                ? (this.displayNext = false)
                : (this.displayNext = true);
              if (this.displayNext) {
                this.displayFirstLevel(1);
                this.createGraph();
              }
            }
          })
          .add(),
        box = text.getBBox();

      chart.renderer
        // .image("https://images.ctfassets.net/hrltx12pl8hq/3MbF54EhWUhsXunc5Keueb/60774fbbff86e6bf6776f1e17a8016b4/04-nature_721703848.jpg?fit=fill&w=480&h=270", box.x - 5, box.y - 5, box.width + 10, box.height + 10)
        .rect(box.x - 5, box.y - 5, box.width + 10, box.height + 10, 5)
        .attr({
          fill: this.displayNext
            ? this.currentPage >
              Math.floor(this.formattedData.length / this.totalElements) - 1
              ? "gray"
              : "#1d5644"
            : "gray",
          stroke: "#1d5644",
          "stroke-width": 1,
          zIndex: 4,
        })
        .add();
    },
    // ciao() {},
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
}
</style>
