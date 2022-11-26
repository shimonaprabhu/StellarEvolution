<template>
  <!-- <button type="button" class="metric">Add Metric</button> -->
  <div class="bee-dd">
    <div class="bee-xaxis">
      Choose X-Axis Parameter:
      <select class="xmetric">
        <option value="Class" selected>Class</option>
        <option value="Temperature">Temperature</option>
      </select>
    </div>
    <div class="bee-yaxis">
      Choose Y-Axis Parameter:
      <select class="ymetric">
        <option value="Magnitude" selected>Magnitude</option>
        <option value="Luminosity">Luminosity</option>
      </select>
    </div>
  </div>
  <div class="toggle">
    Radius:
    <label class="switch">
      <input type="checkbox" />
      <span class="slider round"></span>
    </label>
  </div>

  <div id="bee"></div>
</template>

<script>
import * as d3 from "d3";
import testData from "../../assets/data/startypes.csv";

var xaxis = "Class";
var yaxis = "Magnitude";
var radius = 3;
var flag = false;

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
      const margin = { top: 60, right: 100, bottom: 120, left: 200 };
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
          .attr("class", "axisColor")
          .call(d3.axisTop(xTop).ticks(5));
        svg
          .append("text")
          .attr(
            "transform",
            "translate(" + width / 2 + " ," + (height + 40) + ")"
          )
          .attr("y", -670)
          .style("text-anchor", "middle")
          .text("Spectral Class")
          .attr("fill", "white")
          .attr("stroke", "white");

        const xBottom = d3
          .scaleLinear()
          .domain(d3.extent(data.map((d) => +d.Temperature)))
          .range([width, 0]);

        svg
          .append("g")
          .attr("transform", `translate(0, ${height})`)
          .attr("class", "axisColor")
          .call(d3.axisBottom(xBottom).ticks(5));
        svg
          .append("text")
          .attr(
            "transform",
            "translate(" + width / 2 + " ," + (height + 40) + ")"
          )
          .style("text-anchor", "middle")
          .text("Temperature (K)")
          .attr("fill", "white")
          .attr("stroke", "white");

        const yRight = d3
          .scaleLog()
          .domain(d3.extent(data.map((d) => +d.Luminosity)))
          .range([height, 0]);
        svg.append("g").attr("class", "axisColor").call(d3.axisRight(yRight));
        svg
          .append("text")
          .attr("transform", "rotate(-90)")
          .attr("y", 1020)
          .attr("x", 0 - height / 2)
          .attr("dy", "1em")
          .style("text-anchor", "middle")
          .text("Relative Luminosity (L/L0)")
          .attr("fill", "white")
          .attr("stroke", "white");

        const yLeft = d3
          .scaleLinear()
          .domain(d3.extent(data.map((d) => +d.Magnitude)))
          .range([0, height]);

        svg
          .append("g")
          .attr("transform", "translate(" + width + " ,0)")
          .attr("class", "axisColor")
          .call(d3.axisLeft(yLeft));
        svg
          .append("text")
          .attr("transform", "rotate(-90)")
          .attr("y", -50)
          .attr("x", 0 - height / 2)
          .attr("dy", "1em")
          .style("text-anchor", "middle")
          .text("Absolute Magnitude (M)")
          .attr("fill", "white")
          .attr("stroke", "white");

        const color = d3
          .scaleOrdinal()
          .range([
            "#33A1B8",
            "#F3F5E7",
            "#FCFB8F",
            "#FBE426",
            "#FA7806",
            "#FD150B",
            "#FB1108",
            "#ABD6E6",
            "#9AD2E1",
            "#42A1C1",
            "#1C5FA5",
            "#172484",
          ]);
        const starMass = d3.extent(data.map((d) => +d["Radius"]));
        const size = d3.scaleSqrt().domain(starMass).range([3, 30]);

        d3.select(".xmetric").on("change", update);
        d3.select(".ymetric").on("change", update);
        d3.select(".slider").on("click", update_radius);
        update();
        function update() {
          xaxis = d3.select(".xmetric").property("value");
          yaxis = d3.select(".ymetric").property("value");

          const tooltip = d3
            .select(id)
            .append("div")
            .style("opacity", 0)
            .attr("class", "tooltip")
            .style("background-color", "white")
            .style("border", "solid")
            .style("border-width", "1px")
            .style("border-radius", "5px")
            .style("padding", "5px")
            .style("position", "absolute");

          // Tooltip
          const mouseover = function (event, d) {
            tooltip.style("opacity", 1);
          };

          const mousemove = function (event, d) {
            tooltip
              .html(`R: ${color(d.Type)}`)
              .style("left", event.x + "px")
              .style("top", event.y + "px");
          };
          const mouseleave = function (event, d) {
            tooltip.transition().duration(200).style("opacity", 0);
          };

          const dots = svg
            .selectAll(".circle")
            .data(data)
            .join("circle")
            .attr("fill", function (d) {
              return color(d.Type);
            })
            .attr("class", "circle")
            .attr("stroke", "black")
            .attr("r", function (d) {
              if (flag) return size(+d["Radius"]);
              else return 2;
            })
            .on("mouseover", mouseover)
            .on("mousemove", mousemove)
            .on("mouseleave", mouseleave)
            .transition()
            .duration(2000)
            .attr("cx", function (d) {
              if (xaxis == "Class") return xTop(d["Class"]);
              else return xBottom(+d["Temperature"]);
            })
            .attr("cy", function (d) {
              if (yaxis == "Magnitude") return yLeft(+d["Magnitude"]);
              else return yRight(+d["Luminosity"]);
            });
        }

        function update_radius() {
          flag = !flag;

          const dots = svg
            .selectAll(".circle")
            .data(data)
            .join("circle")
            .attr("fill", function (d) {
              return color(d.Type);
            })
            .attr("class", "circle")
            .attr("stroke", "black")
            .transition()
            .duration(2000)
            .attr("r", function (d) {
              if (flag) return size(+d["Radius"]);
              else return 2;
            });
        }
      });
    },
  },
};
</script>

<style>
#bee {
  background-color: #113;
}

.toggle {
  display: flex;
  color: white;
  gap: 10px;
  align-items: center;
}

.axisColor line {
  stroke: white;
}

.axisColor path {
  stroke: white;
}

.axisColor text {
  stroke: white;
}

.bee-dd {
  display: flex;
  gap: 20px;
  color: white;
}
.xmetric {
  background: #113;
  border-color: #d9d9d9;
  border-radius: 2px;
  font-size: 14px;
  padding: 4px 8px;
  height: 32px;
  width: 180px;
  touch-action: manipulation;
  transition: all 0.3s cubic-bezier(0.645, 0.045, 0.355, 1);
  cursor: pointer;
  white-space: nowrap;
  text-align: center;
  line-height: 1.5715;
  position: relative;
  display: inline-block;
  font-family: "Open Sans", sans-serif;
}
.xmetric:hover {
  border-color: #1890ff;
  border-right-width: 1px;
}
.xmetric::selection {
  color: #fff;
  background: #1890ff;
}

.ymetric {
  background: #113;
  border-color: #d9d9d9;
  border-radius: 2px;
  font-size: 14px;
  padding: 4px 8px;
  height: 32px;
  width: 180px;
  touch-action: manipulation;
  transition: all 0.3s cubic-bezier(0.645, 0.045, 0.355, 1);
  cursor: pointer;
  white-space: nowrap;
  text-align: center;
  line-height: 1.5715;
  position: relative;
  display: inline-block;
  font-family: "Open Sans", sans-serif;
}
.ymetric:hover {
  border-color: #1890ff;
  border-right-width: 1px;
}
.ymetric::selection {
  color: #fff;
  background: #1890ff;
}

.switch {
  display: flex;
  position: relative;
  width: 60px;
  height: 34px;
}
.switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  -webkit-transition: 0.4s;
  transition: 0.4s;
}

.slider:before {
  position: absolute;
  content: "";
  height: 26px;
  width: 26px;
  left: 4px;
  bottom: 4px;
  background-color: white;
  -webkit-transition: 0.4s;
  transition: 0.4s;
}

input:checked + .slider {
  background-color: #2196f3;
}

input:focus + .slider {
  box-shadow: 0 0 1px #2196f3;
}

input:checked + .slider:before {
  -webkit-transform: translateX(26px);
  -ms-transform: translateX(26px);
  transform: translateX(26px);
}

.slider.round {
  border-radius: 36px;
}

.slider.round:before {
  border-radius: 50%;
}
</style>
