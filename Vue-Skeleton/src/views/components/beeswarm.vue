<template>
  <!-- <button type="button" class="metric">Add Metric</button> -->
  <div class="sidenav">
    <button class="dropdown-btn">X-Axis</button>

    <select class="xmetric">
      <option value="Class" selected>Class</option>
      <option value="Temperature">Temperature</option>
    </select>

    <button class="dropdown-btn">Y-Axis</button>

    <select class="ymetric">
      <option value="Magnitude" selected>Magnitude</option>
      <option value="Luminosity">Luminosity</option>
    </select>

    <button class="dropdown-btn">Color Scheme</button>

    <select class="color-scheme">
      <!-- <option value="Magnitude" selected>Magnitude</option>
        <option value="Luminosity">Luminosity</option> -->
      <option value="Type">Type</option>
      <option value="Class" selected>Class</option>
    </select>

    <button class="dropdown-btn-radius">
      Radius

      <label style="display: inline-block" class="switch">
        <input type="checkbox" />
        <span class="slider round"></span>
      </label>
    </button>
    <div id="color-class" style="display: none">
      <button class="dropdown-btn">Legend For Class</button>

      <ul>
        <li style="color: rgb(94, 99, 247)">O</li>
        <li style="color: rgb(26, 166, 236)">B</li>
        <li style="color: rgb(144, 225, 239)">A</li>
        <li style="color: rgb(255, 255, 255)">F</li>
        <li style="color: rgb(245, 223, 56)">G</li>
        <li style="color: rgb(247, 133, 2)">K</li>
        <li style="color: rgb(247, 26, 2)">M</li>
      </ul>
    </div>
    <div id="color-type" style="display: none">
      <button class="dropdown-btn">Legend For Type</button>
      <ul>
        <li style="color: rgb(213, 94, 0)">Red Dwarf</li>
        <li style="color: rgb(230, 159, 0)">Brown Dwarf</li>
        <li style="color: rgb(255, 255, 255)">White Dwarf</li>
        <li style="color: rgb(204, 121, 167)">Main Sequence</li>
        <li style="color: rgb(0, 158, 115)">Giant</li>
        <li style="color: rgb(0, 114, 178)">Supergiant</li>
      </ul>
    </div>
  </div>

  <!-- 
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
    <div class="color-sort">
      Choose the Classification Scheme:
      <select class="color-scheme">
        <option value="Magnitude" selected>Magnitude</option>
        <option value="Luminosity">Luminosity</option>
        <option value="Class" selected>Class</option>
        <option value="Temperature">Temperature</option>
      </select>
    </div>
  </div>
  <div class="toggle">
    Radius:
    <label class="switch">
      <input type="checkbox" />
      <span class="slider round"></span>
    </label>
  </div> -->

  <div id="bee"></div>
</template>

<script>
import * as d3 from "d3";
import testData from "../../assets/data/startypes.csv";

import { transition } from "d3-transition";

var xaxis = "Class";
var yaxis = "Magnitude";
var color_scheme = "Type";

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
      const margin = { top: 80, right: 100, bottom: 120, left: 250 };
      const height = 600;
      const width = 1000;

      var dropdown = document.getElementsByClassName("dropdown-btn");
      var i;

      for (i = 0; i < dropdown.length; i++) {
        dropdown[i].addEventListener("click", function () {
          this.classList.toggle("active");
          var dropdownContent = this.nextElementSibling;
          if (dropdownContent.style.display === "block") {
            dropdownContent.style.display = "none";
          } else {
            dropdownContent.style.display = "block";
          }
        });
      }
      function getTransition() {
        return transition().duration(750);
      }

      d3.csv(localData).then(function (data) {
        const svg = d3
          .select(id)
          .append("svg")
          .attr("width", width + margin.left + margin.right)
          .attr("height", height + margin.top + margin.bottom)
          .append("g")
          .attr("transform", `translate(${margin.left},${margin.top})`);

        var ci = ["-1.5", "-1.0", "-0.5", "0.0", "0.5", "1.0", "1.5"];

        const xTop = d3
          .scaleBand()
          .domain(["O", "B", "A", "F", "G", "K", "M"])
          .range([0, width]);

        svg
          .append("g")
          .attr("transform", `translate(0)`)
          .attr("class", "axisxtop")
          .call(
            d3
              .axisTop(xTop)
              .ticks(5)
              .tickFormat((d, i) => ci[i])
          );
        svg
          .append("text")
          .attr(
            "transform",
            "translate(" + width / 2 + " ," + (height + 40) + ")"
          )
          .attr("y", -670)
          .style("text-anchor", "middle")
          .text("Color Index")
          .attr("fill", "white")
          .attr("stroke", "white");

        const xBottom = d3
          .scaleLinear()
          .domain(d3.extent(data.map((d) => +d.Temperature)))
          .range([width, 0]);

        svg
          .append("g")
          .attr("transform", `translate(0, ${height})`)
          .attr("id", "axisxbottom")
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
        svg.append("g").attr("class", "axisyright").call(d3.axisRight(yRight));
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
          .attr("id", "axisyleft")
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

        const color_type = d3.scaleOrdinal().domain([0, 1, 2, 3, 4, 5]).range([
          // "#FD150B",
          // "#c7630c",
          // "#fff",
          // "#FBE426",
          // "#04c963",
          // "#04a5c9"
          "rgb(213,94,0)",
          "rgb(230,159,0)",
          "rgb(255,255,255)",
          "rgb(204,121,167)",
          "rgb(0,158,115)",
          "rgb(0,114,178)",
        ]);
        const color_class = d3
          .scaleOrdinal()
          .domain(["O", "B", "A", "F", "G", "K", "M"])
          .range([
            // "#33A1B8",
            //   "#F3F5E7",
            //   "#FCFB8F",
            //   "#FBE426",
            //   "#FA7806",
            //   "#FD150B",
            //   "#FD150B"
            "rgb(94, 99, 247)",
            "rgb(26, 166, 236)",
            "rgb(144, 225, 239)",
            "rgb(255,255,255)",
            "rgb(245, 223, 56)",
            "rgb(247, 133, 2)",
            "rgb(247, 26, 2)",
          ]);

        const glow_tooltip = d3
        .select(id)
        .append("div")
        .style("opacity", 1)
        .attr("class", "tooltip")
        .style("color", "rgb(220,208,255)")
        .style("font-size", "20px")
        .style("background-color", "#113")
        .style("padding", "5px")
        .style("position", "absolute")
        .style("cursor", "pointer")
        .style("font-style", "italic");

        const starMass = d3.extent(data.map((d) => +d["Radius"]));
        const size = d3.scaleSqrt().domain(starMass).range([3, 30]);

        d3.select(".xmetric").on("change", update);
        d3.select(".ymetric").on("change", update);
        d3.select(".color-scheme").on("change", update);
        d3.select(".slider").on("click", update_radius);
        update();
        function update() {
          // svg.selectAll(".circle").remove();
          xaxis = d3.select(".xmetric").property("value");
          yaxis = d3.select(".ymetric").property("value");
          color_scheme = d3.select(".color-scheme").property("value");

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
              .html(`R: ${color_type(d.Type)}`)
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
              if (color_scheme == "Class") {
                document.getElementById("color-class").style.display = "block";
                document.getElementById("color-type").style.display = "none";
                return color_class(d.Class);
              } else {
                document.getElementById("color-type").style.display = "block";
                document.getElementById("color-class").style.display = "none";
                return color_type(d.Type);
              }
            })
            .attr("class", "circle")
            .attr("stroke", "black")
            .attr("r", function (d) {
              if (flag) return size(+d["Radius"]);
              else return 2;
            })
            // .style("filter","url(#glow)")
            .style("filter", function (d) {
              if ((d["Glow"])!="N")
              return "url(#glow)";
            })
            .on("click", function (event, d) {
            if ((d["Glow"])!="N") {
              console.log("Glow");
              glow_tooltip.html(`Name`)
                
            } 
          })
            .on("mouseover", mouseover)
            .on("mousemove", mousemove)
            .on("mouseleave", mouseleave)
            .transition()
            .duration(2000)
            .attr("cx", function (d) {
              if (xaxis == "Class") return xTop(d["Class"]) + 72;
              else return xBottom(+d["Temperature"]);
            })
            .attr("cy", function (d) {
              if (yaxis == "Magnitude") return yLeft(+d["Magnitude"]);
              else return yRight(+d["Luminosity"]);
            });

          var filter = svg.append("defs").append("filter").attr("id", "glow"),
          feGaussianBlur = filter
            .append("feGaussianBlur")
            .attr("stdDeviation", "5")
            .attr("result", "coloredBlur"),
          feMerge = filter.append("feMerge"),
          feMergeNode_1 = feMerge
            .append("feMergeNode")
            .attr("in", "coloredBlur"),
          feMergeNode_2 = feMerge
            .append("feMergeNode")
            .attr("in", "SourceGraphic");


          // const brush = d3.brush().on("start brush end", brushed);

          // svg.call(brush);

          
          // function brushed(event) {
          //   console.log("brushed");
          //   let value = [];
          //   if (event.selection) {
          //     const [[x0, y0], [x1, y1]] = event.selection;

          //     value = dots
          //       .style("fill", "white")
          //       .filter(function (d) {
          //         if (xaxis == "Temperature" && yaxis == "Magnitude") {
          //           x0 <= xBottom(d.Temperature) &&
          //             xBottom(d.Temperature) < x1 &&
          //             y0 <= yLeft(d.Magnitude) &&
          //             yLeft(d.Magnitude) < y1;
          //         }
          //         // console.log(x0, y0, x1, y1);
          //         // console.log(
          //         //   "X",
          //         //   xBottom(d.Temperature) && xBottom(d.Temperature),
          //         //   "Y",
          //         //   yLeft(d.Magnitude) && yLeft(d.Magnitude)
          //         // );
          //       })
          //       .style("fill", "grey")
          //       .data();
          //     //   .filter(function (d) {
          //     //   // if (xaxis == "Class" && yaxis == "Magnitude") {
          //     //   //   return (d => x0 <= xTop(d.Class) && xTop(d.Class) < x1 && y0 <= yLeft(d.Magnitude) && yLeft(d.Magnitude) < y1);

          //     //   // }
          //     //   // else if (xaxis == "Class" && yaxis == "Luminosity"){
          //     //   //   console.log('2');
          //     //   //   return (d => x0 <= xTop(d.Class) && xTop(d.Class) < x1 && y0 <= yRight(d.Luminosity) && yRight(d.Luminosity) < y1);

          //     //   // }
          //     //    if (xaxis == "Temperature" && yaxis == "Magnitude"){
          //     //     console.log(xBottom(d.Temperature))
          //     //     return (d => x0 <= xBottom(d.Temperature) && xBottom(d.Temperature) < x1 && y0 <= yLeft(d.Magnitude) && yLeft(d.Magnitude) < y1);

          //     //   }
          //     //   // else if (xaxis == "Temperature" && yaxis == "Luminosity"){
          //     //   //   console.log('4');
          //     //   //   return (d => x0 <= xBottom(d.Temperature) && xBottom(d.Temperature) < x1 && y0 <= yRight(d.Luminosity) && yRight(d.Luminosity) < y1);

          //     //   // }
          //     // })
          //   } else {
          //     dots.style("fill", "white");
          //   }
          //   svg.property("value", value).dispatch("input");
          // }
        }

        function update_radius() {
          flag = !flag;

          const dots = svg
            .selectAll(".circle")
            .data(data)
            .join("circle")
            .attr("fill", function (d) {
              if (color_scheme == "Class") return color_class(d.Class);
              else return color_type(d.Type);
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

.axisxtop line {
  stroke: white;
}

.axisxtop path {
  stroke: white;
}

.axisxtop text {
  stroke: white;
}

#axisxbottom line {
  stroke: white;
}

#axisxbottom path {
  stroke: white;
}

#axisxbottom text {
  stroke: white;
}

.axisyright line {
  stroke: white;
}

.axisyright path {
  stroke: white;
}

.axisyright text {
  stroke: white;
}

#axisyleft line {
  stroke: white;
}

#axisyleft path {
  stroke: white;
}

#axisyleft text {
  stroke: white;
}

.bee-dd {
  display: flex;
  gap: 20px;
  color: white;
}
.xmetric {
  background: #113;
  /* border-color: #d9d9d9;
  border-radius: 2px; */
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
  /* border-color: #d9d9d9;
  border-radius: 2px; */
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

.color-scheme {
  background: #113;
  /* border-color: #d9d9d9;
  border-radius: 2px; */
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
.color-scheme:hover {
  border-color: #1890ff;
  border-right-width: 1px;
}
.color-scheme::selection {
  color: #fff;
  background: #1890ff;
}

.switch {
  display: inline-block;
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
  background-color: grey;
}

input:focus + .slider {
  box-shadow: 0 0 1px grey;
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

/* Fixed sidenav, full height */
.sidenav {
  height: 100%;
  width: 180px;
  position: absolute;
  z-index: 1;
  top: 0;
  left: 0;
  background-color: #111;
  overflow-x: hidden;
  padding-top: 20px;
}

/* Style the sidenav links and the dropdown button */
.dropdown-btn {
  padding: 6px 8px 6px 16px;
  text-decoration: none;
  font-size: 20px;
  color: #818181;
  display: block;
  border: none;
  background: none;
  width: 100%;
  text-align: left;
  cursor: pointer;
  outline: none;
}

/* On mouse-over */
.dropdown-btn:hover {
  color: #f1f1f1;
}

.dropdown-btn-radius {
  padding: 6px 8px 6px 16px;
  text-decoration: none;
  font-size: 20px;
  color: #818181;
  border: none;
  background: none;
  width: 100%;
  text-align: left;
  cursor: pointer;
  outline: none;
}

.dropdown-btn-radius:hover {
  color: #f1f1f1;
}

/* Add an active class to the active dropdown button */
/* .active {
  background-color: grey;
  color: white;
} */

/* Dropdown container (hidden by default). Optional: add a lighter background color and some left padding to change the design of the dropdown content */
.xmetric {
  display: none;
  background-color: #5a5a5a;
  border: none;
  padding-left: 8px;
}

.ymetric {
  display: none;
  background-color: #5a5a5a;
  border: none;
  padding-left: 8px;
}

.color-scheme {
  display: none;
  background-color: #5a5a5a;
  border: none;
  padding-left: 8px;
}

/* Some media queries for responsiveness */
@media screen and (max-height: 450px) {
  .sidenav {
    padding-top: 15px;
  }
  .sidenav a {
    font-size: 18px;
  }
}
</style>