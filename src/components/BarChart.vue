<template>
    <h2>201811153031 曾锐</h2>
    <div id="bar-chart-container">
      <p v-for="(char, i) in alphabet" :key="i">
        <span>{{ char.letter }}</span>
        <span>{{ char.frequency }}</span>
      </p>
    </div>
</template>

<script>
import axios from "axios";
import * as d3 from "d3";
export default{
    data(){
        return {
            alphabet: [],
            color:"steelblue",
            margin: { top: 30, right: 0, bottom: 30, left: 40 },
        };
    },
     /**
     * 在挂载后即开始执行
     */
    mounted(){
        axios.get("./test.json").then((res) =>{
            console.log(res.data);
            this.alphabet=res.data;
            const barChartdata=Object.assign(this.formatData(res.data),{
                format:"%",
                y:"↑ Frequency",
            });
            this.drawBarChart(barChartdata);
        });
    },
    methods: {
    /**
     * 格式化数据
     */
    formatData(data) {
      return data
        .map(({ letter, frequency }) => {
          return { name: letter, value: frequency };
        })
        .sort((a, b) => d3.descending(a.value, b.value));
    },

    /**
     * 绘制直方图
     */
    drawBarChart(data) {
      const margin = this.margin;

      const width = 800;
      const height = 500;

      // 初始化 SVG 元素
      const svg = d3
        .select("#bar-chart-container")
        .append("svg")
        .attr("class", "bar-chart")
        .attr("viewBox", `0 0 ${width} ${height}`)
        .attr("width", width)
        .attr("height", height)
        .append("g");

      // https://observablehq.com/@d3/d3-scaleband
      // x 轴的缩放比例尺
      const x = d3
        .scaleBand()
        .domain(d3.range(data.length))
        .range([margin.left, width - margin.right])
        .padding(0.1);

      // y 轴的缩放比例尺
      const y = d3
        .scaleLinear()
        .domain([0, d3.max(data, (d) => d.value)])
        .nice()
        .range([height - margin.bottom, margin.top]);

      // x 坐标轴
      // tickSizeOuter(0) 移除 0 处初始的标记
      // tickFormat https://github.com/d3/d3-scale/blob/master/README.md#tickFormat
      const xAxis = (g) =>
        g.attr("transform", `translate(0,${height - margin.bottom})`).call(
          d3
            .axisBottom(x)
            .tickFormat((i) => data[i].name)
            .tickSizeOuter(0)
        );

      // y 坐标轴
      const yAxis = (g) =>
        g
          .attr("transform", `translate(${margin.left},0)`)
          .call(d3.axisLeft(y).ticks(null, data.format))
          // 移除区域间的竖线
          .call((g) => g.select(".domain").remove())
          .call((g) =>
            g
              .append("text")
              .attr("x", -margin.left)
              .attr("y", 10)
              .attr("fill", "currentColor")
              .attr("text-anchor", "start")
              .text(data.y)
          );

      svg
        .append("g")
        .attr("fill", this.color)
        .selectAll("rect")
        .data(data)
        .join("rect")
        .attr("x", (d, i) => x(i))
        .attr("y", (d) => y(d.value))
        .attr("height", (d) => y(0) - y(d.value))
        .attr("width", x.bandwidth());

      // 绘制到 SVG
      svg.append("g").call(xAxis);
      svg.append("g").call(yAxis);
    },
  },
};


</script>
