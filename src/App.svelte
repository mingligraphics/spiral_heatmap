<script>
    ////https://gist.github.com/larsenmtl/222043d93a41d48b58d2bfa1e3d4f708
//https://observablehq.com/d/47030b399f7e749e
    import data from "./data/precipitation.json";
    import { timeFormat, timeParse } from "d3-time-format";
    import { scaleLinear, scaleBand } from "d3-scale";
    import { schemeGreens } from "d3-scale-chromatic";
    import { arc } from "d3-shape";
    import {max, min} from "d3-array";

  var formatTime_day = timeFormat("%d")
  var day_of_year = timeFormat("%j")
  var formatTime_month = timeFormat("%b")

  data.forEach((d) => {
    d.newDate = timeParse("%Y-%m-%d")(d.newDate);
    d.newDate = new Date(d.newDate);
    d.dayOfYear = (day_of_year(d.newDate) * 1).toString();
    d.dayOfYear = +d.dayOfYear;
  });

  let years = [...new Set(data.map((d) => d.newYear))];
  let axisData = data.filter((d) => d.newYear == 2000 && formatTime_day(d.newDate) == 1);

//   const max_data = max(data, (d) => d["precipitation(mm)"])
//   const min_data = min(data, (d) => d["precipitation(mm)"])

  console.log(data)

  const margin ={
    top: 40,
    right: 30,
    bottom: 40,
    left:30
  }

  $: width = 880
  $: innerWidth = width - margin.left - margin.right;

  let height = 880;
  let innerHeight = height - margin.top - margin.bottom;

  $: shortLen = (width > height ? height : width) < 620
  ? 620
  : width > height
  ? height
  : width

  $: radiusRange = [(shortLen * 0.22) / 2, (shortLen * 0.88) / 2]
   
  $: angleScaleArc = scaleLinear()
  .domain([1, 365])
  .range([0, Math.PI * 2])

  $: angleScaleAxis = scaleLinear()
  .domain([1, 365])
  .range([-Math.PI / 2, 2 * Math.PI - Math.PI / 2])

  $: radiusAxisScale = scaleBand()
  .domain(years)
  .range(radiusRange).padding(0.2)

  let colors = schemeGreens[9];

  console.log(colors[0])

  $: getColors = (amount) => {
  if (amount <= 0.1) {
    return colors[0];
  } else if (amount > 0.1 && amount <= 10) {
    return colors[1];
  } else if (amount > 10 && amount <= 50) {
    return colors[3];
  } else if (amount > 50 && amount <= 100) {
    return colors[5];
  } else if (amount > 100) {
    return colors[8];
  }
}

  $: Path_arc = arc()
        .innerRadius((d) => radiusAxisScale(d.newYear)) 
        .outerRadius((d) => radiusAxisScale(d.newYear) + radiusAxisScale.bandwidth()) 
        .startAngle((d) => angleScaleArc(d.dayOfYear)) 
        .endAngle((d) => angleScaleArc(d.dayOfYear + 1)); 

</script>

<div class="chart-container" bind:clientWidth={width}>
    <svg {width} {height}>
        <g transform="translate({width/2} {height/2})">
        {#each data as point}
        <path
        d={Path_arc(point)} 
        fill={getColors(point["precipitation(mm)"])}
        stroke="black"
        stroke-width=0/>
        {/each}
        {#each axisData as point}
        <line 
        x1={radiusRange[0] * 1 * Math.cos(angleScaleAxis(point.dayOfYear))}
        x2={radiusRange[1] * 1.0 * Math.cos(angleScaleAxis(point.dayOfYear + 0))}
        y1={radiusRange[0] * 1 * Math.sin(angleScaleAxis(point.dayOfYear + 0))}
        y2={radiusRange[1] * 1.0 * Math.sin(angleScaleAxis(point.dayOfYear + 0))}
        stroke-width="1"
        stroke="#222"
        stroke-dasharray="2,2"/>
        <text
        class="month_label"
        x={radiusRange[1] * 1.05 * Math.cos(angleScaleAxis(point.dayOfYear + 15))}
        y={radiusRange[1] * 1.05 * Math.sin(angleScaleAxis(point.dayOfYear + 15))}
        >
        {formatTime_month(point.newDate)}
        </text>
       {/each}
       {#each [2000, 2010, 2020] as point}
       <text
       class="year_label"
       x=3
       y={-radiusAxisScale(point)}>
        {point}
       </text> 
       {/each}
        </g>
    </svg>
</div>

<style>
    .month_label{
      fill: #666;
      text-anchor: start;
      font-family: Retina,  sans-serif;
      font-size: 15px;
      line-height: 15.6px;
    }
    .year_label{
      fill: #666;
      text-anchor: start;
      font-family: RetinaNarrowLight,  sans-serif;
      font-size: 13px;
      line-height: 15.6px;
    }
</style>
