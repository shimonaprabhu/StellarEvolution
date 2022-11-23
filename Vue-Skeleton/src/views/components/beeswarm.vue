<template>
  <button type="button" class="metric">Add Metric</button>
  <div id="bee"></div>
</template>

<script>
import * as d3 from "d3";
import testData from "../../assets/data/startypes.csv";
// import testData from "../../assets/data/flare.csv";

export default {
  name: "BeeChart",
  props: {
    myBeechartData: Array,
  },
  mounted() {
    try {
      console.log(testData);
      let localData = testData;
      this.drawBeeChart(localData, "#bee");
      console.log("Data Passed down as a Prop  ");
    } catch (e) {
      console.log("MOUNT ERROR", e);
    }
  },
  methods: {
    drawBeeChart(localData, id) {
      const margin = { top: 40, right: 100, bottom: 120, left: 200 };
      const height = 600;
      const width = 1000;
      d3.csv(localData).then(function (data) {
        const svg = d3
          .select(id)
          .append("svg")
          .attr("width", width + margin.left + margin.right)
          .attr("height", height + margin.top + margin.bottom)
          .append("g")
          .attr("transform", `translate(${margin.left},${margin.top})`);

        const xTop = d3
          .scaleBand()
          .domain(["O", "B", "A", "F", "G", "K", "M"])
          .range([0, width]);

        svg
          .append("g")
          .attr("transform", `translate(0)`)
          .call(d3.axisTop(xTop).ticks(5));
        svg
          .append("text")
          .attr(
            "transform",
            "translate(" + width / 2 + " ," + (height + 40) + ")"
          )
          .attr("y", -670)
          .style("text-anchor", "middle")
          .style("font-weight", "bold")
          .text("Spectral Class");

        const xBottom = d3
          .scaleLinear()
          .domain(d3.extent(data.map((d) => +d.Temperature)))
          .range([width, 0]);

        svg
          .append("g")
          .attr("transform", `translate(0, ${height})`)
          .call(d3.axisBottom(xBottom).ticks(5));
        svg
          .append("text")
          .attr(
            "transform",
            "translate(" + width / 2 + " ," + (height + 40) + ")"
          )
          .style("text-anchor", "middle")
          .style("font-weight", "bold")
          .text("Temperature (K)");

        const yRight = d3
          .scaleLog()
          .domain(d3.extent(data.map((d) => +d.Luminosity)))
          .range([height, 0]);
        svg.append("g").call(d3.axisRight(yRight));
        svg
          .append("text")
          .attr("transform", "rotate(-90)")
          .attr("y", 1020)
          .attr("x", 0 - height / 2)
          .attr("dy", "1em")
          .style("text-anchor", "middle")
          .style("font-weight", "bold")
          .text("Luminosity (L/L0)");

        const yLeft = d3
          .scaleLinear()
          .domain(d3.extent(data.map((d) => +d.Magnitude)))
          .range([0, height]);

        svg
          .append("g")
          .attr("transform", "translate(" + width + " ,0)")
          .call(d3.axisLeft(yLeft));
        svg
          .append("text")
          .attr("transform", "rotate(-90)")
          .attr("y", -50)
          .attr("x", 0 - height / 2)
          .attr("dy", "1em")
          .style("text-anchor", "middle")
          .style("font-weight", "bold")
          .text("Absolute Magnitude (M)");

        const color = d3.scaleOrdinal().range(d3.schemePaired);

        const starMass = d3.extent(data.map((d) => +d["Radius"]));
        const size = d3.scaleSqrt().domain(starMass).range([3, 30]);

        svg
          .selectAll(".circle")
          .data(data)
          .enter()
          .append("circle")
          .attr("class", "circle")
          .attr("stroke", "black")
          .attr("fill", (d) => color(d.Type))
          .attr("r", (d) => size(+d["Radius"]))
          //   .attr("r", (d) => 5)
          .attr("cx", (d) => xBottom(+d.Temperature))
          .attr("cy", (d) => yLeft(+d.Magnitude));
      });
    },
  },
};
</script>

<style></style>
