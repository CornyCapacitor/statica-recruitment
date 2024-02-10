<script setup lang="ts">
import { ref, onMounted } from 'vue'
import {
  SciChartSurface,
  NumericAxis,
  FastLineRenderableSeries,
  XyDataSeries,
  EllipsePointMarker,
  SweepAnimation,
  SciChartJsNavyTheme,
  NumberRange,
  MouseWheelZoomModifier,
  ZoomPanModifier,
  ZoomExtentsModifier,
  EAutoRange,
  FastColumnRenderableSeries
} from "scichart";

type Data = {
  dt: number,
  price: number,
  amount: number,
}

const fetchedData = ref([]);

const fetchData = async () => {
  try {
    const res = await fetch("https://rest.statica.pl/rest/stockquotes/gpw:PLKGHM000017?type=trades&dt_from=2010-01-01&limit=10000",{
      headers: {
        'Authorization': 'Basic ZnJvbnRlbmQyMDI0OnRlc3Q='
    }}
    );
    const data = await res.json();
    fetchedData.value = data
    
    return data;
  } catch (error) {
    console.error("Error fetching the data:", error)
  }
}

const initSciChart = async (data: Data[]) => {

  const parsedData = [];
  
  for(let entry of data){
    const last = parsedData[parsedData.length - 1];

    if(parsedData.lenght ===0){
      parsedData.push(entry);
    }
    else if(last && last.dt === entry.dt){
      last.amount += entry.amount;
    } else {
      parsedData.push(entry)
    }
  }

  SciChartSurface.useWasmFromCDN();
  const { sciChartSurface, wasmContext } = await SciChartSurface.create("scichart-root", {
    titleStyle: { fontSize: 22 }
  });

  const growByX = new NumberRange(1, 1);
  const growByY = new NumberRange(1, 1);
  sciChartSurface.xAxes.add(new NumericAxis(wasmContext, { growByX,
    visibleRange: new NumberRange(2, 2000),
  }));
  sciChartSurface.yAxes.add(new NumericAxis(wasmContext, { growByY,
   }));


  sciChartSurface.renderableSeries.add(new FastLineRenderableSeries(wasmContext, {
    stroke: "steelblue",
    strokeThickness: 1,
    dataSeries: new XyDataSeries(wasmContext, {
      xValues: data.map((x,i)=>i),
      yValues: data.map(x=> x.price)
    }),
    pointMarker: new EllipsePointMarker(wasmContext, { width: 1, height: 1, fill: "#fff" }),
    animation: new SweepAnimation({ duration: 300, fadeEffect: true })
  }));

  sciChartSurface.renderableSeries.add(new FastColumnRenderableSeries(wasmContext, {
    stroke: "red",
    strokeThickness: 1,
    dataSeries: new XyDataSeries(wasmContext, {
      xValues: data.map((x,i)=>i),
      yValues: data.map(x=> x.amount)
    }),
    pointMarker: new EllipsePointMarker(wasmContext, { width: 1, height: 1, fill: "#fff" }),
  }));

  sciChartSurface.chartModifiers.add(
    new MouseWheelZoomModifier(), 
    new ZoomPanModifier(), 
    new ZoomExtentsModifier()
  );
};

onMounted(() => {
  fetchData().then(data =>{

initSciChart(data);
  })
})


</script>

<template>
  <div>
    <h1>Statica Recruitment</h1>
    <div id="scichart-root"></div>
  </div>
</template>

<style scoped>
#scichart-root{
  width: 800px;
  height: 800px;
}

</style>
