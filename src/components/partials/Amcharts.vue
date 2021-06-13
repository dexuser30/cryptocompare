<template>
    <div class="app__amcharts">
        <div class="app__amcharts-title">
            <div class="h1">{{selectedPair.pair2}} | {{selectedPair.pair1}}</div>
        </div>

        <div
            v-show="loaded"
            class="app__amchart"
            ref="chartdiv"
        ></div>

        <div v-show="!loaded" class="loader">
            <div id="spinner"></div>
        </div>
    </div>
</template>

<script>
import axios from 'axios'

import * as am4core from "@amcharts/amcharts4/core"
import * as am4charts from "@amcharts/amcharts4/charts"
import am4themes_animated from "@amcharts/amcharts4/themes/animated"

am4core.useTheme(am4themes_animated)

export default {
    name: "Amcharts",
    props: ['compareData', 'apiKey'],
    data: () => ({
        chart: null,
        limit: 10,
        loaded: false,
        loading: false,
        isCanUpdateChart: true
    }),
    computed: {
        selectedPair() {
            let filteredData = this.compareData.pairs.filter(item => item.id === this.compareData.selectedPair)
            filteredData = filteredData[0]

            this.setChart(filteredData)

            return filteredData
        }
    },
    watch: {
        'selectedPair.data.P': {
            handler: function () {
                if(this.isCanUpdateChart) {
                    this.isCanUpdateChart = false

                    setTimeout(() => {
                        this.isCanUpdateChart = true
                    }, 5 * 1000)
                }
            },
        },
    },
    methods: {
        async setChart(pairData) {
            if(!this.isCanUpdateChart) return

            let responseData

            await axios
                .get(`https://min-api.cryptocompare.com/data/v2/histominute?fsym=${pairData.pair1}&tsym=${pairData.pair2}&limit=${this.limit}&api_key=${this.apiKey}`)
                .then((response) => {
                    responseData = response.data.Data
                })
                .finally(() => {
                    this.loaded = true
                })

            let chartData = []

            responseData.Data.forEach((item) => {
                let date = new Date(item.time * 1000)
                let value = item.close

                chartData.push({date, value})
            })

            this.chart.data = chartData
        }
    },
    mounted() {
        this.chart = am4core.create(this.$refs.chartdiv, am4charts.XYChart)

        this.chart.paddingRight = 20;

        this.chart.xAxes.push(new am4charts.DateAxis())

        let valueAxis = this.chart.yAxes.push(new am4charts.ValueAxis());
        valueAxis.tooltip.disabled = true;
        valueAxis.renderer.minWidth = 35;

        let series = this.chart.series.push(new am4charts.LineSeries());
        series.dataFields.dateX = "date";
        series.dataFields.valueY = "value";

        series.tooltipText = "{valueY.value}";
        this.chart.cursor = new am4charts.XYCursor();

        let scrollbarX = new am4charts.XYChartScrollbar();
        scrollbarX.series.push(series);
        this.chart.scrollbarX = scrollbarX;
    },
    beforeDestroy() {
        if (this.chart) {
            this.chart.dispose();
        }
    }
}
</script>