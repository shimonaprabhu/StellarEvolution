<template>
  <!-- <button type="button" class="metric">Add Metric</button> -->
  <div class="heading"> Stellar Evolution </div>
  <div class="sidenav">
    <div class="title">Control Panel</div>
    <br />
    <div class="starinfo">Star Information</div>
    <br />
    <div class="panel">
      <div class="starinfo">Filters</div>
      <br />
      <!-- <button class="reset">Reset</button> -->
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

      <!-- <button class="dropdown-btn">Group</button>

      <select class="group">
        <option value="All Groups" selected>All Groups</option>
        <option value="Red Dwarf">Red Dwarf</option>
        <option value="Brown Dwarf">Brown Dwarf</option>
        <option value="White Dwarf">White Dwarf</option>
        <option value="Main Sequence">Main Sequence</option>
        <option value="Giant">Giant</option>
        <option value="Supergiant">Supergiant</option>

      </select> -->

      <button class="dropdown-btn-radius">
        Group

        <label style="display: inline-block" class="switch">
          <input type="checkbox" />
          <span class="slider2 round"></span>
        </label>
      </button>

      <button class="dropdown-btn-radius">
        Radius

        <label style="display: inline-block" class="switch">
          <input type="checkbox" />
          <span class="slider1 round"></span>
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
          <li style="color: #FC6A0C">Red Dwarf</li>
          <li style="color: #FFF601">Brown Dwarf</li>
          <li style="color: rgb(255, 255, 255)">White Dwarf</li>
          <li style="color: rgb(204, 121, 167)">Main Sequence</li>
          <li style="color: #00CC66">Giant</li>
          <li style="color: #00BFFF">Supergiant</li>
        </ul>
      </div>
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
      <span class="slider1 round"></span>
    </label>
  </div> -->

  <div class="reset-btn">
    <button class="reset">Reset</button>
  </div>

  <div id="bee"></div>
</template>

<script>
import * as d3 from "d3";
import testData from "../../assets/data/startypes.csv";

import { transition } from "d3-transition";

var xaxis = "Class";
var yaxis = "Magnitude";
var color_scheme = "Type";
var group_name = "All Groups"

var radius = 3;
var flag = false;
var flag2= false;

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
      const margin = { top: 40, right: 100, bottom: 120, left: 250 };
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
          // .domain(d3.extent(data.map((d) => +d.Temperature)))
          .domain([0, 42000])
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
          // .domain(d3.extent(data.map((d) => +d.Luminosity)))
          .domain([0.00002, 8888888])
          .range([height, 0]);
        svg.append("g").attr("class", "axisyright").call(d3.axisRight(yRight));
        svg
          .append("text")
          .attr("transform", "rotate(-90)")
          .attr("y", -50)
          .attr("x", 0 - height / 2)
          .attr("dy", "1em")
          .style("text-anchor", "middle")
          .text("Relative Luminosity (L/L0)")
          .attr("fill", "white")
          .attr("stroke", "white");

        const yLeft = d3
          .scaleLinear()
          // .domain(d3.extent(data.map((d) => +d.Magnitude)))
          .domain([-14, 21])
          .range([0, height]);

        svg
          .append("g")
          .attr("transform", "translate(" + width + " ,0)")
          .attr("id", "axisyleft")
          .call(d3.axisLeft(yLeft));

        svg
          .append("text")
          .attr("transform", "rotate(-90)")
          .attr("y", 1020)
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
          "#FC6A0C",
          "#FFF601",
          "rgb(255,255,255)",
          "rgb(204,121,167)",
          "#00CC66",
          "#00BFFF"
        ]);
        const color_class = d3
          .scaleOrdinal()
          .domain(["O", "B", "A", "F", "G", "K", "M"])
          .range([
            "rgb(94, 99, 247)",
            "rgb(26, 166, 236)",
            "rgb(144, 225, 239)",
            "rgb(255,255,255)",
            "rgb(245, 223, 56)",
            "rgb(247, 133, 2)",
            "rgb(247, 26, 2)",
          ]);

        const starMass = d3.extent(data.map((d) => +d["Radius"]));
        const size = d3.scaleSqrt().domain(starMass).range([3, 30]);

        const glow_tooltip = d3
          .select(id)
          .append("div")
          .style("color", "white")
          .style("opacity", 0)
          .attr("class", "tooltip")
          .style("width", "180px")
          .style("font-size", "16px")
          .style("position", "absolute")
          .style("left", 18 + "px")
          .style("top", 130 + "px");

        d3.select(".xmetric").on("change", update);
        d3.select(".ymetric").on("change", update);
        d3.select(".color-scheme").on("change", update);
        // d3.select(".group").on("change", update_group);
        d3.select(".slider1").on("click", update_radius);
        d3.select(".slider2").on("click", update_group);
        // d3.select(".reset").on("click", update);
        update();
        function update() {
          // svg.selectAll(".color-scheme").remove();
          xaxis = d3.select(".xmetric").property("value");
          yaxis = d3.select(".ymetric").property("value");
          color_scheme = d3.select(".color-scheme").property("value");
          

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

          function dragged(event, d) {
            d3.select(this).attr("cx", event.x).attr("cy", event.y);
          }

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
            .style("filter", function (d) {
              if (d["Glow"] != "N") return "url(#glow)";
            })
            .on("click", function (event, d) {
              if (d["Glow"] != "N") {
                glow_tooltip.style("opacity", 1);
                glow_tooltip.html(
                  `<p style="padding-bottom:2px;">Name: ${d.Glow}</p> <p style="padding-bottom:2px;">Temperature: ${d.Temperature}K</p><p style="padding-bottom:2px;">Radius: ${d.Radius}x</p>`
                );
              } else glow_tooltip.style("opacity", 0);
            })
            .call(d3.drag().on("drag", dragged))
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

          //   svg
          //     .selectAll(".circle")
          //     .data(data)
          //     .join("circle")
          //     .call(d3.drag().on("drag", dragged));
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

        function update_group() {
          flag2 = !flag2;
          
          if (!flag2){

              svg.selectAll("*").remove();
        
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
          // .domain(d3.extent(data.map((d) => +d.Temperature)))
          .domain([0, 42000])
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
          // .domain(d3.extent(data.map((d) => +d.Luminosity)))
          .domain([0.00002, 8888888])
          .range([height, 0]);
        svg.append("g").attr("class", "axisyright").call(d3.axisRight(yRight));
        svg
          .append("text")
          .attr("transform", "rotate(-90)")
          .attr("y", -50)
          .attr("x", 0 - height / 2)
          .attr("dy", "1em")
          .style("text-anchor", "middle")
          .text("Relative Luminosity (L/L0)")
          .attr("fill", "white")
          .attr("stroke", "white");

        const yLeft = d3
          .scaleLinear()
          // .domain(d3.extent(data.map((d) => +d.Magnitude)))
          .domain([-14, 21])
          .range([0, height]);

        svg
          .append("g")
          .attr("transform", "translate(" + width + " ,0)")
          .attr("id", "axisyleft")
          .call(d3.axisLeft(yLeft));

        svg
          .append("text")
          .attr("transform", "rotate(-90)")
          .attr("y", 1020)
          .attr("x", 0 - height / 2)
          .attr("dy", "1em")
          .style("text-anchor", "middle")
          .text("Absolute Magnitude (M)")
          .attr("fill", "white")
          .attr("stroke", "white");
              update()
          }
          else{
          const brush = d3.brush()
          .on("start brush end", brushed);
          svg.call(brush);

          function brushed({selection}) {
              console.log(selection)
          }
          }

          

          }



      });
    },
  },
};
</script>

<style>
#bee {
  background-color: rgb(0, 7, 29);
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

.heading {
  text-align: center;
  font-size: 36px;
  color: white;
  padding-left: 150px;
  text-shadow: 0px 0px 30px white;
}

.xmetric {
  background: rgb(0, 7, 29);
  /* border-color: #d9d9d9;
  border-radius: 2px; */
  font-size: 16px;
  height: 32px;
  width: 180px;
  touch-action: manipulation;
  transition: all 0.3s cubic-bezier(0.645, 0.045, 0.355, 1);
  cursor: pointer;
  white-space: nowrap;
  line-height: 1.5715;
  position: relative;
  display: inline-block;
  font-family: "Open Sans", sans-serif;
  
  padding: 1px 1px;
  padding: 6px 8px 6px 16px;
  color: white;
  background-color: rgb(0, 7, 29);
  display: none;
  border: none;
  padding-left: 10px;
  text-align: center;
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
  background: rgb(0, 7, 29);
  /* border-color: #d9d9d9;
  border-radius: 2px; */
  font-size: 16px;
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

  padding: 1px 1px;
  padding: 6px 8px 6px 16px;
  color: white;
  background-color: rgb(0, 7, 29);
  display: none;
  border: none;
  padding-left: 10px;
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
  background: rgb(0, 7, 29);
  /* border-color: #d9d9d9;
  border-radius: 2px; */
  font-size: 16px;
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

  padding: 1px 1px;
  padding: 6px 8px 6px 16px;
  color: white;
  background-color: rgb(0, 7, 29);
  display: none;
  border: none;
  padding-left: 10px;
}
.color-scheme:hover {
  border-color: #1890ff;
  border-right-width: 1px;
}
.color-scheme::selection {
  color: #fff;
  background: #1890ff;
}

.group {
  background: rgb(0, 7, 29);
  /* border-color: #d9d9d9;
  border-radius: 2px; */
  font-size: 16px;
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

  padding: 1px 1px;
  padding: 6px 8px 6px 16px;
  color: white;
  background-color: rgb(0, 7, 29);
  display: none;
  border: none;
  padding-left: 10px;
}
.group:hover {
  border-color: #1890ff;
  border-right-width: 1px;
}
.group::selection {
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

.slider1 {
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

.slider1:before {
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

input:checked + .slider1 {
  background-color: grey;
}

input:focus + .slider1 {
  box-shadow: 0 0 1px grey;
}

input:checked + .slider1:before {
  -webkit-transform: translateX(15px);
  -ms-transform: translateX(15px);
  transform: translateX(15px);
}

.slider1.round {
  border-radius: 24px;
  height: 1.5em;
  width: 2.5em;
}

.slider1.round:before {
  border-radius: 50%;
  padding-top: 10px;
  height: 1em;
  width: 1em;
}

.slider2 {
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

.slider2:before {
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

input:checked + .slider2 {
  background-color: grey;
}

input:focus + .slider2 {
  box-shadow: 0 0 1px grey;
}

input:checked + .slider2:before {
  -webkit-transform: translateX(15px);
  -ms-transform: translateX(15px);
  transform: translateX(15px);
}

.slider2.round {
  border-radius: 24px;
  height: 1.5em;
  width: 2.5em;
}

.slider2.round:before {
  border-radius: 50%;
  padding-top: 10px;
  height: 1em;
  width: 1em;
}

/* Fixed sidenav, full height */
.sidenav {
  height: 100%;
  width: 180px;
  position: absolute;
  z-index: 1;
  top: 0;
  left: 0;
  background-color: rgb(0, 7, 29);
  border-right: 1px solid white;
  overflow-x: hidden;
  padding-top: 20px;
}

.title {
  text-align: center;
  color: white;
  font-size: 20px;
  /* border-bottom: 0.1px solid white;
  border-top: 0.1px solid white; */
}

.starinfo {
  padding-top: 7px;
  padding-left: 10px;
  text-align: center;
  color: white;
  text-transform: uppercase;
}

.panel {
  padding-top: 150px;
}

/* Style the sidenav links and the dropdown button */
.dropdown-btn {
  padding: 6px 8px 6px 16px;
  text-decoration: none;
  font-size: 16px;
  color: white;
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
  color: #818181;
}

.dropdown-btn-radius {
  padding: 6px 8px 0px 16px;
  text-decoration: none;
  font-size: 16px;
  color: white;
  border: none;
  background: none;
  width: 100%;
  text-align: left;
  cursor: pointer;
  outline: none;
}

.dropdown-btn-radius:hover {
  color: #818181;
}

/* Add an active class to the active dropdown button */
/* .active {
  background-color: grey;
  color: white;
} */

/* Dropdown container (hidden by default). Optional: add a lighter background color and some left padding to change the design of the dropdown content */


.tooltip {
  z-index: 1;
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
