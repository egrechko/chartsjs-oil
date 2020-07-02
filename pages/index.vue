<template>
  <div class="section">
    <div class="container">
      <h1 class="title is-size-1">US Oil Stocks</h1>
      <hr />

      <div class="section">
        <h2 class="is-size-2">Last 3 Years - Monthly</h2>
        <!-- chart goes here -->
        <canvas id="chart" ref="chart" width="800" height="400"></canvas>
      </div>
    </div>
  </div>
</template>

<script>
// import dayjs from 'dayjs'
import moment from 'moment'
import Chart from 'chart.js'

export default {
  async asyncData({ $axios }) {
    // pulling in data from eia.gov
    // eslint-disable-next-line
    const oilStocks = await $axios
      .$get('http://api.eia.gov/series/', {
        params: {
          api_key: process.env.EIA_API_KEY,
          series_id: 'PET.MCRSTUS1.M',
        },
      })
      .then(function (res) {
        // returning only the data not the whole request
        return res.series[0].data
      })

    oilStocks.forEach((element) => {
      // fixing the date of each piece of data recieved.
      element[0] = element[0].replace(/(\d{4})(\d{2})/, `$1-$2`)
    })

    // set the data in Vue.
    return { oilStocks }
  },
  data() {
    return {
      oil3Year: {},
    }
  },
  created() {
    // create a list of data from the last 3 years
    this.oil3Year.all = this.oilStocks.filter((element) => {
      if (moment(element[0]).isAfter(moment().subtract(3, 'year'))) {
        return element
      }
    })

    // now sort the data from oldest first for chart js. The chart starts at the oldest date and moves towards the latest piece of data.
    this.oil3Year.all = this.oil3Year.all.sort(function (a, b) {
      return moment(a[0]).subtract(moment(b[0]))
    })
  },
  mounted() {
    // setup the chart
    const ctx = this.$refs.chart

    // eslint-disable-next-line
    const myChart = new Chart(ctx, {
      type: 'line',
      data: {
        labels: [], // here we add the dates
        datasets: [
          {
            label: 'US Oil Inventory',
            data: [], // here we add the amount built
          },
        ],
      },
      options: {
        scales: {
          xAxes: [
            {
              type: 'time',
              time: {
                unit: 'month',
              },
            },
          ],
        },
      },
    })

    // add the data
    this.oil3Year.all.forEach((element) => {
      // add the label
      myChart.data.labels.push(element[0])
      // add the data
      myChart.data.datasets[0].data.push(element[1])
    })

    // update the chart
    myChart.update()
  },
}
</script>
