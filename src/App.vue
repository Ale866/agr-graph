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
      slicedData: [],
      totalElements: 10,
      mainNode: {
        name: "AGRICOLUS",
        id: 273,
        level: 0,
      },
      previousNode: "",
      storedData: [],
      currentPage: 0,
      prova: true,
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
                "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI2NiIsIm5hbWUiOiJtLmZvY2FyZXRhQGFncmljb2x1cy5jb20iLCJyb2xlIjpbIkFkbWluaXN0cmF0b3IiLCJBcHByb3ZlcnMiLCJDYW5DaGFuZ2VGZWF0dXJlcyIsIkNhbkFkZEZlYXR1cmVzIl0sIm9yZ2FuaXphdGlvbl9pZCI6IjY1IiwidXNlcl9maXN0X25hbWUiOiJNaWNoZWxlIiwidXNlcl9sYXN0X25hbWUiOiJGb2NhcmV0YSIsImFncm9fb2YiOiIxODEiLCJ0b2tlbl91c2FnZSI6ImFjY2Vzc190b2tlbiIsImp0aSI6IjM4MzA2MTg2LTMxMDgtNGQ1OS1iNGUzLWMyNWVhZTlmOTI3ZiIsInNjb3BlIjpbIm9wZW5pZCIsInByb2ZpbGUiLCJlbWFpbCJdLCJhenAiOiJhZ3JpY29sdXNfd2ViIiwibmJmIjoxNjMyMjk1NjU0LCJleHAiOjE2MzIzMzg4NTQsImlhdCI6MTYzMjI5NTY1NCwiaXNzIjoiaHR0cHM6Ly9zdGFnaW5nLWF1dGguYWdyaWNvbHVzLmNvbS8ifQ.hyBE5SYYLiXHPMjQ9Kf8PmIZsdXSCEKhQKX1oLxIxeg",
            },
          }
        )
        .then((resp) => {
          this.getDataResponse = resp.data.filter(
            (f) => !(f.type == "Customer" && f.parentId == "273")
          );
        });
    },
    displayFirstLevel(n: number) {
      for (let i = 0; i < this.getDataResponse.length; i++) {
        if (this.getDataResponse[i].level === 1) {
          this.formattedData[i] = [
            this.mainNode.name,
            this.getDataResponse[i].name,
          ];
        }
      }

      this.formattedData = this.formattedData.sort((a, b) =>
        a[1].toUpperCase().localeCompare(b[1].toUpperCase())
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
        if (this.getDataResponse[i].name === event.point.name)
          var currentNode = this.getDataResponse[i];
      }
      if (event.point.name === "AGRICOLUS") {
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
      let index = 1;
      this.slicedData = [];
      let level = node.level + 1;
      let flag = false;
      if (node.parentId === this.mainNode.id) {
        this.storedData[0] = [this.mainNode.name, node.name];
        console.log("AAAAAA");
      }
      if (this.storedData.length > 0) {
        this.slicedData[0] = this.storedData[0];
      }
      let prevNode = this.previousNode;

      if (prevNode != node) {
        let response = this.getDataResponse.filter(
          (f) => f.level == level && f.parentId == node.id
        );
        for (let i = 0; i < response.length; i++) {
          let r = response[i];
          if (prevNode && !flag) {
            this.slicedData[index] = [prevNode.name, node.name];
            index++;
          }
          this.slicedData[index] = [node.name, r.name];
          index++;
          flag = true;
        }
      }
      if (node.id == 273) {
        this.slicedData = this.slicedData.slice(1, this.totalElements + 1);
      }
      if (flag) {
        this.previousNode = node;
      }
      if (this.slicedData.length > 1) {
        this.createGraph();
      }
    },
    displayNode(node, i) {
      // se nodo di indice i + 1 ha lo stesso id del nodo di indice i, diplay uno sotto l'altro
      let body = document.getElementsByTagName("body")[0];
      if (i === 0) {
        node.plotX = body.clientWidth / 2;
        node.plotY = 70;
      } else if (i !== 0 && this.prova == true) {
        node.plotX = ((body.clientWidth - 140) / this.totalElements) * i;
        node.plotY = 300;
        node.color =
          this.getDataResponse[i].type == "BusinessUnit" ? "#362dcc" : "yellow";
      }
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
            formatter: function () {
              let dummyArray = [];
              dummyArray = that.getDataResponse;
              dummyArray.unshift(that.mainNode);
              const t = dummyArray.find((e) => e.name == this.key);
              let string;
              if (t.email != undefined) {
                string = `<strong>email:</strong> ${t.email} <br> <strong>name:</strong> ${t.name} <br> <strong>id:</strong> ${t.id}`;
              } else {
                string = `<strong>name:</strong> ${t.name} <br> <strong>id:</strong> ${t.id}`;
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
                symbol: "circle",
                radius: 15,
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
                  // var chart = this.series[0].chart
                  // width = chart.plotWidth,
                  // height = chart.plotHeight;

                  this.nodes.forEach(function (node, i) {
                    that.displayNode(node, i);
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
                    symbol: "circle",
                    radius: 30,
                    fillColor: "green",
                  },
                },
              ],
              id: "chart",
              data: this.slicedData,
            },
          ],
        },
        function (chart): void {
          var point = chart.series[0],
            text = chart.renderer
              .text(
                "Next",
                (point.plotX = body.clientWidth - 60),
                (point.plotY = 350)
              )
              .attr({
                zIndex: 5,
              })
              .on("click", function () {
                that.displayFirstLevel(1);
                that.createGraph();
              })
              .add(),
            box = text.getBBox();

          chart.renderer
            .rect(box.x - 5, box.y - 5, box.width + 10, box.height + 10, 5)
            .attr({
              fill:
                that.currentPage >
                Math.floor(that.formattedData.length / that.totalElements) - 1
                  ? "gray"
                  : "green ",
              stroke: "green",
              "stroke-width": 1,
              zIndex: 4,
            })
            .add();

          (text = chart.renderer
            .text(
              "Prev",
              (point.plotX = body.clientWidth - (body.clientWidth - 30)),
              (point.plotY = 350)
            )
            .attr({
              zIndex: 5,
            })
            .on("click", function () {
              that.displayFirstLevel(-1);
              that.createGraph();
            })
            .add()),
            (box = text.getBBox());

          chart.renderer
            .rect(box.x - 5, box.y - 5, box.width + 10, box.height + 10, 5)
            .attr({
              fill: that.currentPage == 0 ? "gray" : "green",
              stroke: "green",
              "stroke-width": 1,
              zIndex: 4,
            })
            .add();
        }
      );
    },
  },
  async mounted() {
    await this.getData();
    this.displayFirstLevel();
    this.createGraph();
  },

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

			let prevNode = this.previousNode;
			if (prevNode != node) {
				let response = this.getDataResponse.filter(f => f.level == level && f.parentId == node.id);
				for (let i = 0; i < response.length; i++) {
					let r = this.response[i];
					if (prevNode && !flag) {
						this.formattedData[index] = [prevNode.name, node.name];
						index++;
					}
					this.formattedData[index] = [node.name, r.name];
					index++;
					flag = true;
				}
			}

			console.log(this.formattedData);

			if (flag) {
				this.previousNode = node;
			}
			if (this.formattedData.length) {
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

	width: 100vw;
	height: 100vh;
	background-color: rgb(198, 223, 192);

path,
text {
	cursor: pointer;
}
</style>
