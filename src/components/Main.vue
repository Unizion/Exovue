<template>
  <div class="hello left">
    <div class="left" v-if="request">
        <div class="bloc" v-for="(item, index) in result.data.list" :key="index">
            <div class="content">
                <div class="cityname">{{ item.name }}</div>
                <div class="temp">
                    {{ parseInt(item.main.temp) }} ° C
                    <div class="tempfeel">ressenti : {{ parseInt(item.main.feels_like) }} °C</div>
                    <div class="description">{{ item.weather[0].description }}</div>
                </div>
                <div class="infos">
                    <div class="tempminmax">
                        <div class="tempmin"><div>{{ parseInt(item.main.temp_min) }}°</div></div>
                        <div class="tempmax"><div>{{ parseInt(item.main.temp_max) }}°</div></div>
                    </div>
                </div>
                <div class="moreinfos">
                    <div class="humidity">Humidité : {{ item.main.humidity }} %</div>
                    <div class="sunrise">Levée du soleil :{{ getHour(item.sys.sunrise) }}</div>
                    <div class="sunset">Couchée du soleil :{{ getHour(item.sys.sunset) }}</div>
                </div>
            </div>
        </div>
        <svg width=960 height=600></svg>
        
    </div>
    
  </div>
  
</template>

<script>

import * as d3 from "d3";
import axios from "axios";

export default {
  name: 'Main',
  props: {
    msg: String
  },
  data() {
    return {
        request: false,
        result: null
    }
  },
  methods: {
    getHour(unixtime){
        var date = new Date(unixtime * 1000);
        var hours = date.getHours();
        var minutes = "0" + date.getMinutes();
        var seconds = "0" + date.getSeconds();
        var time = hours + ':' + minutes.substr(-2) + ':' + seconds.substr(-2);
        return time;
    }
  },
  mounted(){
  
    //d3 Chart
    let render = (data) => {
        let svg_width = 960;
        let svg_height = 600;
        let chart_width = 840;
        //let chart_height = 480;
        let list = data.data.list
        let cities = [];
        for (let x = 0; x < list.length; x ++){
            cities.push(list[x].name.toString())
        }
        let svg = d3
            .select("svg")
            .attr("width",svg_width)
            .attr("height",svg_height);
            
        this.chart = svg
            .append("g")
            .attr("transform",`translate(60,60)`);
            
        let xScale = d3.scaleBand()
            .domain(cities)
            .range([0,chart_width])
            .padding(0.2)
        let xAxis = d3.axisBottom(xScale)
            
            
        let yScale = d3
            .scaleLinear()
            .domain([0,20])
            .range([480,0])
        let yAxis = d3.axisLeft(yScale).ticks(5)
            
        this.chart
            .append("g")
            .attr('transform','translate(0,0)')
            .call(yAxis);
            
        this.chart
            .append("g")
            .attr("transform",`translate(0,480)`)
            .call(xAxis);
            
        this.chart = svg.append("g")
            .attr('transform',`translate(60,60)`)
        
        
        let barGroups = this.chart
            .selectAll(".rect")
            .data(list.map(((item) => parseInt(item.main.temp))))
            .enter()
        
        let rectPadding = 20
        
        barGroups
            .append("rect")
            .attr("x", function(d,i) {
                return xScale(cities[i]);
              })
            .attr("y", g => yScale(g))
            .attr("width", () => (xScale.step() - rectPadding))
            .attr("height", g => 480 - yScale(g))
            .attr('fill', 'lightblue')
            
        barGroups.append('text')
            .attr('x', function (d, i) {
                return (xScale(cities[i])+ ((xScale.step() / 2 ) - 40));
            })
            .attr("y", g => yScale(g) - 20)
            .attr("class", "charttemps")
            .text(function (d){ return d + " °C" } ) 
    }
    axios
      .get("https://api.openweathermap.org/data/2.5/group?id=2988507,2643743,2950159,2661552,3117735,2761369,2800866,4219762&lang=fr&units=metric&appid=39dd26102ee402322ecaac1e630e5997")
      .then(response => (this.result = response))
      .then(() => this.request = true)
      .then(() => render(this.result))
  }

  
}
</script>

<style scoped>

.left {
    float: left;
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

.bloc{
    color:white;
    background-color: rgba(150,150,150,0.6);
    width: 45%;
    margin-right: 2.5%;
    margin-left: 2.5%;
    margin-bottom: 50px;
    display: inline-flex;
    border-radius: 10px;
}
.bloc .content{
    width:100%;
    padding: 25px;
}
.bloc .cityname{
    font-size : 45px;
    font-weight: bold;
    float:left
}
.bloc .temp{
    font-size: 45px;
    float:right;
    font-weight: bold;
    color: white;
    letter-spacing: 2px;
    padding-left:20px;
    border-left : 2px solid black;
    text-align:right;
}
.bloc .temp .tempfeel {
    font-size: 20px;
    margin-bottom: 20px;
}
.bloc .temp .description {
    text-transform: capitalize;
    font-size:20px;
}

.bloc .infos{
    float: right;
    padding: 0 25px;
}
.bloc .infos .tempmin ,.bloc .infos .tempmax {
    font-size: 20px;
    margin-bottom: 15px;
    height: 50px;
    width: 50px;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius:50%;
    background-color: rgba(255,255,255,0.6);
}
.bloc .infos .tempmin {
    color:#6e61e2;
}
.bloc .infos .tempmax{
    color:#cc6c5f;
}
.bloc .infos .humidity {
    font-size: 25px;
    height: 30px;
}
.bloc .moreinfos{
    float: right;
    padding: 0px 25px;
    text-align: right;
    color:grey;
    font-weight: bold;
}
.bloc .moreinfos .sunrise, .bloc .moreinfos .sunset,.bloc .moreinfos .humidity{
    padding-bottom:15px;
}
.bloc .moreinfos .humidity{
    padding-top:15px;
}
</style>
