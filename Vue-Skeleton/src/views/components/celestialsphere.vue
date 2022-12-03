<template>
<div class="heading-sphere"> Celestial Sphere </div>
  <div id="sphere"></div>
</template>

<script>
import * as d3 from "d3";
import stars from "../../assets/data/stars.6.json";
import starNames from "../../assets/data/starnames.json";
import constellationLines from "../../assets/data/constellations.lines.json";
import constellations from "../../assets/data/constellationName.json";

export default {
  name: "SphereChart",
  props: {
    mySpherechartData: Array,
  },
  mounted() {
    try {
      const localData = "hi";
      this.drawSphereChart(localData, "#sphere");
      console.log("Data Passed down as a Prop  ");
    } catch (e) {
      console.log("MOUNT ERROR", e);
    }
  },
  methods: {
    drawSphereChart(localData, id) {
      const margin = { top: 40, right: 100, bottom: 120, left: 250 };
      const height = "600px";
      const width = "1100px";
      const padding = 10;
      const sensitivity = 75;
      const starlist = [
        "Sirius",
        "Betelgeuse",
        "Rigel",
        "Canopus",
        "Arcturus",
        "Antares",
        "Capella",
        "Deneb",
        "Aldebaran",
        "Altair",
        "Pollux",
        "Castor",
        "Bellatrix",
        "Procyon",
        "Vega",
        "Polaris",
        "Proxima Centauri",
      ];

      const starIDs = [
        32349, 27989, 24436, 30438, 69673, 80763, 24608, 102098, 21421, 97649,
        37826, 36850, 25336, 37279, 91262, 11767, 68191,
      ];

      let starObj = {
        Sirius: {
          age: "230 Million",
          current: "Main Sequence",
          future: "Red Giant -> Planetary Nebula  -> White Dwarf",
        },
        Betelgeuse: {
          age: "10 Million",
          current: "Red Supergiant",
          future: "Supernova -> Neutron Star/Black Hole",
        },
        Rigel: {
          age: "8 Million",
          current: "White Supergiant",
          future: "Supernova -> Neutron Star/Black Hole",
        },
        Canopus: {
          age: "10 Million",
          current: "Main Sequence",
          future: "Planetary Nebula -> White Dwarf",
        },
        Arcturus: {
          age: "7.1 Billion",
          current: "Red Giant",
          future: "Planetary Nebula -> White Dwarf",
        },
        Antares: {
          age: "11.01 Million",
          current: "Red Supergiant",
          future: "Supernova -> Neutron Star/Black Hole",
        },
        Capella: {
          age: "400 Million",
          current: "Red Giant",
          future: "Evolving Red Giant",
        },
        Deneb: {
          age: "10.01 Million",
          current: "Blue Supergiant",
          future: "Supernova -> Neutron Star/Black Hole",
        },
        Aldebaran: {
          age: "6.6 Billion",
          current: "Red Giant",
          future: "Evolving Red Giant",
        },
        Altair: {
          age: "1.2 Billion",
          current: "Main Sequence",
          future: "Red Giant -> Planetary Nebula -> White Dwarf",
        },
        Pollux: {
          age: "724.5 Million",
          current: "Red Giant",
          future: "Planetary Nebula -> White Dwarf",
        },
        Castor: {
          age: "370 Million",
          current: "Main Sequence",
          future: "Red Giant -> Planetary Nebula -> White Dwarf",
        },
        Bellatrix: {
          age: "25 Million",
          current: "Main Sequence",
          future: "Giant -> Supergiant -> Supernova",
        },
        Procyon: {
          age: "1.7 Billion",
          current: "Main Sequence",
          future: "Red Giant -> Planetary Nebula -> White Dwarf",
        },
        Vega: {
          age: "455.3 Million",
          current: "Main Sequence",
          future: "Red Giant -> Planetary Nebula -> White Dwarf",
        },
        Polaris: {
          age: "70 Million",
          current: "Supergiant",
          future: "Supernova -> Neutron Star/Black Hole",
        },
        "Proxima Centauri": {
          age: "4.85 Billion",
          current: "Red Dwarf",
          future: "Main Sequence -> Blue Dwarf",
        },
      };

      const svg = d3
        .select(id)
        .append("svg")
        .attr("width", width)
        .attr("height", height)
        .attr("transform", `translate(${margin.left},${margin.top})`);

      const projection = d3["geoOrthographic"]().precision(0.1);
      const initialScale = projection.scale();

      const magnitudeScale = d3
        .scaleLinear()
        .domain(d3.extent(stars.features, (d) => d.properties.mag))
        .range([3, 0.5]);

      const starPath = d3.geoPath(projection).pointRadius(function (d) {
        if (starIDs.includes(d.id)) return 5;
        else return magnitudeScale(d.properties.mag);
      });

      const constellationLinesPath = d3.geoPath(projection);

      const tooltip_stars = d3
        .select(id)
        .append("div")
        .style("opacity", 0)
        .attr("class", "tooltip")
        .style("color", "rgb(220,208,255)")
        .style("font-size", "16px")
        .style("background-color", "rgb(0, 7, 29)")
        .style("padding", "5px")
        .style("position", "absolute")
        .style("cursor", "pointer")
        .style("font-style", "italic");

      const tooltip_constellation = d3
        .select(id)
        .append("div")
        .style("opacity", 0)
        .attr("class", "tooltip")
        .style("color", "rgb(136,205,236)")
        .style("font-size", "16px")
        .style("background-color", "rgb(0, 7, 29)")
        .style("padding", "5px")
        .style("position", "absolute")
        .style("cursor", "pointer")
        .style("font-style", "italic");

      const starFuture = d3
        .select(id)
        .append("div")
        .style("color", "white")
        .style("opacity", 0)
        .attr("class", "tooltip")
        .style("font-size", "16px")
        .style("background-color", "rgb(0, 7, 29)")
        .style("padding", "10px")
        .style("padding-top", "120px")
        .style("height", "100%")
        .style("width", "180px")
        .style("position", "absolute")
        .style("left", 0 + "px")
        .style("top", 800 + "px")
        .style("border-right","1px solid white");
      
      const sidenav = d3
        .select(id)
        .append("div")
        .attr("class", "sidenav")
        .style("color", "white")
        .style("font-size", "16px")
        .style("background-color", "rgb(0, 7, 29)")
        .style("padding", "10px")
        .style("height", "100%")
        .style("width", "180px")
        .style("position", "absolute")
        .style("left", "0px")
        .style("top", "800px");

      // Tooltip
      const mouseover_stars = function (event, d) {
        tooltip_stars.style("opacity", 1);
      };

      const mouseover_constellation = function (event, d) {
        tooltip_constellation.style("opacity", 1);
      };

      const mousemove_stars = function (event, d) {
        if (starNames[d.id].name) {
          tooltip_stars
            .html(`${starNames[d.id].name}`)
            .style("left", 820 + "px")
            .style("top", 1250 + "px");
        }
      };
      const mousemove_constellation = function (event, d) {
        for (let i = 0; i < 89; i++) {
          if (constellations.features[i].id == d.id) {
            tooltip_constellation
              .html(
                `${constellations.features[i].properties.name}`
              )
              .style("left", 820 + "px")
              .style("top", 1250 + "px");
          }
        }
      };
      const mouseleave_stars = function (event, d) {
        tooltip_stars.transition().duration(200).style("opacity", 0);
      };
      const mouseleave_constellation = function (event, d) {
        tooltip_constellation.transition().duration(200).style("opacity", 0);
      };

      svg
        .append("g")
        .attr("id", "constellation")
        .selectAll("path")
        .data(constellationLines.features)
        .enter()
        .append("path")
        .attr("d", constellationLinesPath)
        .attr("fill", "rgb(0, 7, 29)")
        .attr("stroke", "#ddd")
        .attr("stroke-width", "0.5px")
        .attr("transform", `translate(100,50)`)
        .on("mouseover", mouseover_constellation)
        .on("mousemove", mousemove_constellation)
        .on("mouseleave", mouseleave_constellation);

      // svg
      //   .selectAll("text")
      //   .data(constellationLines.features)
      //   .enter()
      //   .append("text")
      //   .text(function (d) {
      //     return d.id;
      //   })
      //   .attr("font_family", "sans-serif") // Font type
      //   .attr("font-size", "11px") // Font size
      // .attr("fill", "darkgreen");

      svg
        .append("g")
        .attr("id", "star")
        .selectAll("path")
        .data(stars.features)
        .enter()
        .append("path")
        .attr("d", starPath)
        .attr("fill", function (d) {
          if (d.id == 32349) return "rgb(204,121,167)";
          if (d.id == 27989) return "#00BFFF";
          if (d.id == 24436) return "#00BFFF";
          if (d.id == 30438) return "rgb(204,121,167)";
          if (d.id == 69673) return "#00CC66";
          if (d.id == 80763) return "#00BFFF";
          if (d.id == 24608) return "#00CC66";
          if (d.id == 102098) return "#00BFFF";
          if (d.id == 21421) return "#00CC66";
          if (d.id == 97649) return "rgb(204,121,167)";
          if (d.id == 37826) return "#00CC66";
          if (d.id == 36850) return "rgb(204,121,167)";
          if (d.id == 25336) return "rgb(204,121,167";
          if (d.id == 37279) return "rgb(204,121,167)";
          if (d.id == 91262) return "rgb(204,121,167)";
          if (d.id == 11767) return "#00BFFF";
          if (d.id == 68191) return "#FC6A0C";
          else return "#fff";
        })
        .on("mouseover", mouseover_stars)
        .on("mousemove", mousemove_stars)
        .on("mouseleave", mouseleave_stars)
        .attr("transform", `translate(100,50)`)
        .on("click", function (event, d) {
          if (starlist.includes(starNames[d.id].name)) {
            // console.log(d);
            sidenav.style("opacity", 0);
            starFuture.style("opacity", 1);
            starFuture
              .html(`Name: ${starNames[d.id].name} <br/><br/> Current Age: ${starObj[starNames[d.id].name].age} <br/><br/> Magnitude: ${d.properties.mag} <br/><br/> B-V Color Index: ${d.properties.bv} <br/><br/> Current Stage: ${starObj[starNames[d.id].name].current} <br/><br/>  Predicted Future: ${starObj[starNames[d.id].name].future}`)
              
          } 
          else if (starNames[d.id].name)
          {
            sidenav.style("opacity", 0);
            starFuture.style("opacity", 1);
            starFuture
              .html(`Name: ${starNames[d.id].name}`)
              
          }
          else{
            sidenav.style("opacity", 1);
            starFuture.style("opacity", 0);
          }
        });

      svg
        .call(
          d3.drag().on("drag", function (event) {
            var rotate = projection.rotate();
            var k = sensitivity / projection.scale();
            projection.rotate([
              rotate[0] + event.dx * k,
              rotate[1] - event.dy * k,
            ]);
            svg.selectAll("path").attr("d", constellationLinesPath);
            svg.selectAll("path").attr("d", starPath);
          })
        )
        .call(
          d3.zoom().on("zoom", (event) => {
            if (event.transform.k > 0.3) {
              projection.scale(initialScale * event.transform.k);
              // path = d3.geoPath().projection(projection);
              svg.selectAll("path").attr("d", starPath);
              svg.attr("r", projection.scale());
            } else {
              event.transform.k = 0.3;
            }
          })
        );
    },
  },
};
</script>

<style>

.heading-sphere {
  text-align: center;
  font-size: 36px;
  color: white;
  padding-left: 270px;
  text-shadow: 0px 0px 30px white;
}

.sidenav-sky {
  height: 100%;
  width: 180px;
  background-color: rgb(0, 7, 29);
}
</style>
