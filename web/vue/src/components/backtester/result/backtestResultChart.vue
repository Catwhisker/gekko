<template lang='jade'>
div
  div#resultHighchart
</template>

<script>

import _ from 'lodash'
import Vue from 'vue'
import * as Highcharts from 'highcharts/highstock'

const colors = ["#7cb5ec", "#90ed7d", "#f7a35c", "#e4d354", "#2b908f", "#f45b5b", "#91e8e1", "#8085e9", "#f15c80"];

export default {
  props: ['result'],
  components: {
    // spinner
  },
  data: () => {
    return {
      chart: null,
    };
  },
  // mixins: [ dataset ],
  methods: {
    // humanizeDuration: (n) => {
    //   return window.humanizeDuration(n, {largest: 4});
    // },
    humanizeTimestamp: timestamp => moment.utc(timestamp).format('YYYY-MM-DD HH:mm'),

    render: function() {
    },
    remove: function() {
      this.chart.destroy();
      //console.log('remove()')
    }
  },
  beforeDestroy: function() {
    this.remove();
  },
  mounted () {
    console.log('backtest result chart mounted', this.result);
    const chartOptions = {
      chart: {
        renderTo: 'resultHighchart',
        zoomType: 'x',
        height: 720,
        ignoreHiddenSeries: true
      },
      rangeSelector: {
        selected: 1
      },
      title: {
        // text: `${this.dataset.asset}-${this.dataset.currency} Trades`
      },
      legend: {
          enabled: true,
          align: 'center',
          backgroundColor: '#FFFFFF',
          borderColor: 'black',
          borderWidth: 0,
          layout: 'horizontal',
          verticalAlign: 'bottom',
          y: 0,
          shadow: false,
          floating: false
      },

      rangeSelector: {
          allButtonsEnabled: true,
          buttons: [{
              type: 'day',
              count: 1,
              text: '1d'
          }, {
              type: 'week',
              count: 1,
              text: '7d'
          }, {
              type: 'month',
              count: 1,
              text: '1m'
          }, {
              type: 'month',
              count: 3,
              text: '3m'
          }, {
              type: 'year',
              count: 1,
              text: '1y'
          }, {
              type: 'ytd',
              count: 1,
              text: 'YTD'
          }, {
              type: 'all',
              text: 'ALL'
          }],
          selected: 6,
          inputEnabled: true,
          enabled: true
      },

      yAxis: [{
        id: 'trade',
        opposite: false,
        height: '70%',
        title: {
          text: `${this.result.report.asset}-${this.result.report.currency} Trades`,
          style: {
            color: colors[0]
          }
        }
      }, {
        id: 'volume',
        gridLineWidth: 0,
        opposite: false,
        top: '70%',
        height: '10%',
        offset: 2,
        title: {
          text: 'volume',
          // style: {
          //   color: color
          // }
        }
      }, {
        id: 'oscillator',
        gridLineWidth: 0,
        opposite: false,
        top: '80%',
        height: '20%',
        offset: 2,
        title: {
          text: 'Oscillator',
          // style: {
          //   color: color
          // }
        }
      }],

      series: [{
        name: `${this.result.report.asset}-${this.result.report.currency}`,
        id: 'trades',
        type: 'candlestick',
        yAxis: 'trade',
        data: this.result.candles.map((candle) => [moment(candle.start).unix() * 1000, candle.open, candle.high, candle.low, candle.close]),
        //color: colors[0],
        //lineWidth: 1,
        tooltip: {
          valueDecimals: 2
        }
      }, {
        type: 'column',
        yAxis: 'volume',
        //name: `${this.result.report.asset}-${this.result.report.currency}`,
        name: 'volume',
        data: this.result.candles.map((candle) => [moment(candle.start).unix() * 1000, candle.volume]),
      }],

      plotOptions: {
        series: {
          animation: false,
          states: {
            hover: {
               lineWidth: 1,
               lineWidthPlus: 1
            }
          }
        //   marker: {
        //     enabled: true
        //   }
        }
      }
    }

    // Add indicator results to chart
    let indicatorResultSeriesId;
    const color = colors[chartOptions.yAxis.length];
    const yAxisId = `y${chartOptions.yAxis.length}`;
    const seriesId = `s${chartOptions.yAxis.length}`;
    indicatorResultSeriesId = seriesId;

    var length = this.result.indicatorResults.length - 1;

    if(length > -1) {
      for(let yAxisName in this.result.indicatorResults[length].result) {
        let yAxisData = this.result.indicatorResults[length].result[yAxisName];
        for(let seriesName in yAxisData) {
          //console.log('seriesName: ', seriesName);

          chartOptions.series.push({
            name: seriesName,
            id: seriesName,
            data: this.result.indicatorResults.map((res) => [moment(res.date).unix() * 1000, res.result[yAxisName][seriesName] ]),
            yAxis: `${yAxisName}`,
            lineWidth: 2,
            //color: color,
            tooltip: {
              valueDecimals: 3
            }
          });
        }
      }
    }

    // Add Buy/Sell Flags
    const buySellFlags = this.result.trades.map((trade) => {
      const isBuy = trade.action === 'buy';
      let text = isBuy ? `Buy for ${trade.price}` : `Sell for ${trade.price}`
      return {
        x: moment(trade.date).unix() * 1000,
        title: isBuy ? 'B' : 'S',
        text,
        marker:{ fillColor: 'red'},
        style: { fontSize: '8px' },
      }
    })
    const flagSeriesId = `f${chartOptions.yAxis.length}`;
    chartOptions.series.push({
      type: 'flags',
      name: 'Flags',
      id: flagSeriesId,
      data: buySellFlags,
      onSeries: 'trades',
      // onSeries: indicatorResultSeriesId,
      // color: stratInstance.highcharts.color,
      // fillColor: stratInstance.highcharts.color,
      shape: 'circlepin',
      width: 8
    });

    //console.log(`${flagSeriesId} is added in the chart.`);

    this.chart = Highcharts.stockChart(chartOptions)
    //console.log('Chart has been created.');

  },
  watch: {
    // setIndex: function() {
    //   this.set = this.datasets[this.setIndex];
    //   this.updateCustomRange();
    //   this.emitSet(this.set);
    // },

    // customTo: function() { this.emitSet(this.set); },
    // customFrom: function() { this.emitSet(this.set); },
    // importCandleSize: function() { this.emitSet(this.set); },
    // importCandleSizeUnit: function() { this.emitSet(this.set); }
  }
}
</script>
<style>
td.radio {
  width: 45px;
}
td label{
  display: inline;
  font-size: 1em;
}
</style>
