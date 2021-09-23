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
			btn: false,
			temp: Object,
			displayNext: true,
			displayPrev: true
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
								"Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI2NiIsIm5hbWUiOiJtLmZvY2FyZXRhQGFncmljb2x1cy5jb20iLCJyb2xlIjpbIkFkbWluaXN0cmF0b3IiLCJBcHByb3ZlcnMiLCJDYW5DaGFuZ2VGZWF0dXJlcyIsIkNhbkFkZEZlYXR1cmVzIl0sIm9yZ2FuaXphdGlvbl9pZCI6IjY1IiwidXNlcl9maXN0X25hbWUiOiJNaWNoZWxlIiwidXNlcl9sYXN0X25hbWUiOiJGb2NhcmV0YSIsImFncm9fb2YiOiIxODEiLCJ0b2tlbl91c2FnZSI6ImFjY2Vzc190b2tlbiIsImp0aSI6IjI5MjQ1NzcyLTMzNTktNDdmZi04Y2JmLTQzZDc2MWNlZTVlNCIsInNjb3BlIjpbIm9wZW5pZCIsInByb2ZpbGUiLCJlbWFpbCJdLCJhenAiOiJhZ3JpY29sdXNfd2ViIiwibmJmIjoxNjMyMzgyMzIwLCJleHAiOjE2MzI0MjU1MjAsImlhdCI6MTYzMjM4MjMyMCwiaXNzIjoiaHR0cHM6Ly9zdGFnaW5nLWF1dGguYWdyaWNvbHVzLmNvbS8ifQ.o7DgzwQlXIcBIBZuq0Z3eir54PY--mVjJYcbxICAtxc",
						},
					}
				)
				.then((resp) => {
					this.getDataResponse = resp.data.filter(
						(f) => !(f.type == "Customer" && f.parentId == "273")
					);
					// for (let i = 0; i < this.getDataResponse.length; i++) {
					// 	console.log(this.getDataResponse[i].id);
					// }

				});
		},
		displayFirstLevel(n?: number) {
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
			let prevNode = this.previousNode;
			let diff = prevNode.level - node.level

			if (node.parentId == this.mainNode.id) {
				this.storedData[0] = [this.mainNode.name, node.name];
			}

			if (this.storedData.length) {
				this.slicedData[0] = this.storedData[0]
			}

			let response = this.getDataResponse.filter(
				(f) => f.level == level && f.parentId == node.id
			);

			if (response.length && prevNode != node) {

				if (prevNode.id == node.parentId) {
					this.storedData.push([prevNode.name, node.name])
				}

				if (diff > 1) {
					while (diff != 1) {
						this.storedData.pop()
						diff--
					}
				}

				for (let i = 1; i <= this.storedData.length; i++) {
					this.slicedData.push(this.storedData[i]);
				}

			}
			index = this.storedData.length

			if (node != this.mainNode) {
				this.prova = false
				this.displayNext = false
				this.displayPrev = false
			}
			else {
				this.prova = true
				this.displayNext = true
				this.displayPrev = true
			}

			this.temp = node

			if (prevNode != node) {
				
				for (let i = 0; i < response.length; i++) {

					let r = response[i];
					// if (prevNode && !flag) {
					// 	this.slicedData[index] = [prevNode.name, node.name];
					// 	index++;
					// }
					this.slicedData[index] = [node.name, r.name];
					index++;
					flag = true;
				}
			}

			if (node.id == 273) {
				this.slicedData = this.slicedData.sort((a, b) =>
					a[1].toUpperCase().localeCompare(b[1].toUpperCase())
				);
				this.slicedData = this.slicedData.slice(0, this.totalElements);
			}
			if (flag) {
				this.previousNode = node;
			}

			if (this.slicedData.length > 1) {
				this.createGraph();
			}
		},
		displayNode(node, i) {
			let body = document.getElementsByTagName("body")[0];
			if (i == 0) {
				node.plotX = body.clientWidth / 2;
				node.plotY = 70;
			} else if (i != 0) {
				var t = this.getDataResponse.find((e) => e.name == node.id)
				// a è node, t è oggetto corrente
				// if (node.id == this.temp.name) {
				// 	var a = node
				// }
				let parentElement = this.getDataResponse.find(e => e.id == this.temp.parentId)
				// if(parentElement){
				// 	console.log(parentElement.name);	
				// }
				if (this.getDataResponse.find((e) => e.id == this.temp.parentId)) {

					// console.log("AAAAA");

				}
				node.color = t.type == "BusinessUnit" ? "#362dcc" : "yellow";
				if (this.prova == true) {
					node.plotX = ((body.clientWidth - 140) / this.totalElements) * i;
					node.plotY = 300;
				}
				else {
					if (i == 1) {
						node.plotX = body.clientWidth / 2;
						node.plotY = 300
					}
					else {

						node.plotX = ((body.clientWidth - 140) / this.totalElements) * i;
						node.plotY = 450;
					}
					// else if (t.id === this.getDataResponse[i - 1].id) {
					// 	node.plotX = ((body.clientWidth - 140) / this.totalElements) * i;
					// 	node.plotY = 450;
					// }
					// let storedNode = node
				}
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
								string = `<strong>email:</strong> ${t.email} <br> <strong>name:</strong> ${t.name} <br> <strong>id:</strong> ${t.id} <br> <strong>parentId:</strong> ${t.parentId}`;
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

									// eslint-disable-next-line @typescript-eslint/no-this-alias
									let nodesThis = this
									this.nodes.forEach(function (node, i) {
										that.displayNode(node, i,);
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
								if (that.displayNext) {
									that.currentPage >
										Math.floor(that.formattedData.length / that.totalElements) - 1
										? that.displayNext = false
										: that.displayNext = true
									if (that.displayNext) {
										that.displayFirstLevel(1);
										that.createGraph();
									}
								}
							})
							.add(),
						box = text.getBBox();

					chart.renderer
						// .image('https://www.highcharts.com/samples/graphics/sun.png', (body.clientWidth - 60), 330, 30, 30)
						.rect(box.x - 5, box.y - 5, box.width + 10, box.height + 10, 5)
						.attr({
							fill:
								that.displayNext ? that.currentPage > Math.floor(that.formattedData.length / that.totalElements) - 1 ? "gray" : "green" : "gray",
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
							if (that.displayPrev) {
								that.currentPage == 0 ? that.displayPrev = false : that.displayPrev = true
								if (that.displayPrev) {
									console.log("A");

									that.displayFirstLevel(-1);
									that.createGraph();
								}
							}

						})
						.add()),
						(box = text.getBBox());

					chart.renderer
						.rect(box.x - 5, box.y - 5, box.width + 10, box.height + 10, 5)
						.attr({
							fill: that.displayPrev ? that.currentPage == 0 ? "gray" : "green" : "gray",
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
path,
text {
	cursor: pointer;
}
</style>
