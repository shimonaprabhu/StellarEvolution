<template>
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
      const margin = { top: 40, right: 10, bottom: 120, left: 200 };
      const height = 500;
      const width = 960;
      d3.csv(localData).then(function (data) {
        const svg = d3
          .select(id)
          .append("svg")
          .attr("width", width + margin.left + margin.right)
          .attr("height", height + margin.top + margin.bottom)
          .append("g")
          .attr("transform", `translate(${margin.left},${margin.top})`);

        const x = d3
          .scaleBand()
          .domain(["O", "B", "A", "F", "G", "K", "M"])

          .range([0, width]);

        svg
          .append("g")
          .attr("transform", `translate(0, ${height})`)
          .call(d3.axisBottom(x).ticks(5));
        svg
          .append("text")
          .attr(
            "transform",
            "translate(" + width / 2 + " ," + (height + 40) + ")"
          )
          .style("text-anchor", "middle")
          .style("font-weight", "bold")
          .text("Spectral Class");
        const y = d3
          .scaleLog()
          .domain(d3.extent(data.map((d) => +d.Luminosity)))
          .range([height, 0]);
        svg.append("g").call(d3.axisLeft(y));
        svg
          .append("text")
          .attr("transform", "rotate(-90)")
          .attr("y", 0 - 50)
          .attr("x", 0 - height / 2)
          .attr("dy", "1em")
          .style("text-anchor", "middle")
          .style("font-weight", "bold")
          .text("Luminosity (L/L0)");

        const color = d3.scaleOrdinal().range(d3.schemePaired);

        const starMass = d3.extent(data.map((d) => +d["Radius"]));
        const size = d3.scaleSqrt().domain(starMass).range([3, 40]);

        svg
          .selectAll(".circle")
          .data(data)
          .enter()
          .append("circle")
          .attr("class", "circle")
          .attr("stroke", "black")
          .attr("fill", (d) => color(d.Type))
          .attr("r", (d) => size(+d["Radius"]))
          .attr("cx", (d) => x(d.Class))
          .attr("cy", (d) => y(d.Luminosity));
      });
    },
  },
};
</script>

<style></style>
