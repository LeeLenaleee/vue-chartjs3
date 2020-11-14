# vue-chartjs-wrapper
A wrapper around the chart.js library based on vue-chart.js

## using the wrapper
You can import Chart and use it as a normal component in your vue app.

For specific documentation about the library see: https://www.chartjs.org/

### example
    <template>
        <Chart :data="data" :options="options" :type="type" /> 
    </template>
    
    <script>
    import Chart from 'vue-chartjs3'
       
    export default {
        components: {
            Chart
        },
        data () {
            return{
                type: 'line',
                data: {
                    labels: [],
                    datasets: [],
                }
                options: {}
            }
        }
    }
    </script>

