<script>
import * as ChartJs from 'chart.js'

for (let chartJsKey in ChartJs) {
  if (!ChartJs[chartJsKey].id) {
    continue;
  }
  ChartJs.Chart.register(ChartJs[chartJsKey])
}

export default {
  props: {
    type: { type: String, default: 'line'},
    data: { required: true },
    options: { required: true }
  },

  watch: {
    data: {
      deep: true,
      handler (newData, oldData) {
        if (!oldData || !this._chart) {
          this.destroy();
          this.render();

          return;
        }

        let chart = this._chart;

        // Get new and old DataSet Labels
        let newDatasetLabels = newData.datasets.map((dataset) => (dataset.label));
        let oldDatasetLabels = oldData.datasets.map((dataset) => (dataset.label));

        // Stringify 'em for easier compare
        const oldLabels = JSON.stringify(oldDatasetLabels);
        const newLabels = JSON.stringify(newDatasetLabels);

        if (oldLabels !== newLabels || oldData.datasets.length !== newData.datasets.length) {
          this.destroy();
          this.render();

          return;
        }

        newData.datasets.forEach((dataset, i) => {
          // Get new and old dataset keys
          const oldDatasetKeys = Object.keys(oldData.datasets[i]);
          const newDatasetKeys = Object.keys(dataset);

          // Get keys that aren't present in the new data
          const deletionKeys = oldDatasetKeys.filter((key) => (key !== '_meta' && newDatasetKeys.indexOf(key) === -1));

          // Remove outdated key-value pairs
          deletionKeys.forEach((deletionKey) => {
            delete chart.data.datasets[i][deletionKey];
          });

          // Update attributes individually to avoid re-rendering the entire chart
          for (const attribute in dataset) {
            if (Object.prototype.hasOwnProperty.call(dataset, attribute)){
              chart.data.datasets[i][attribute] = dataset[attribute];
            }
          }
        });

        if (Object.prototype.hasOwnProperty.call(newData, 'labels')){
          chart.data.labels = newData.labels;
          this.$emit('labels:update');
        }
        if (Object.prototype.hasOwnProperty.call(newData, 'xLabels')){
          chart.data.xLabels = newData.xLabels;
          this.$emit('xlabels:update');
        }
        if (Object.prototype.hasOwnProperty.call(newData, 'yLabels')){
          chart.data.yLabels = newData.yLabels;
          this.$emit('ylabels:update');
        }

        this.update();
      }
    },

    type () {
      if (!this._chart) {
        throw new Error('No chart!');
      }

      this.render();
    },

    options: {
      deep: true,
      handler () {
        if (!this._chart) {
          throw new Error('No chart!');
        }

        this._chart.options = this.options;
        this.update();
      }
    },
  },

  methods: {
    render () {
      this.destroy();
      if (!this.$refs.canvas) {
        throw new Error('Initialisation error, no canvas available');
      }

      this._chart = new ChartJs.Chart(this.$refs.canvas.getContext('2d'), {
        type: this.type,
        data: this.data,
        options: this.options
      });

      this.$emit('chart:render');
    },

    update () {
      if (!this._chart) {
        throw new Error('No chart!');
      }

      this._chart.update();

      this.$emit('chart:update');
    },

    destroy () {
      if (!this._chart) {
        return;
      }

      this._chart.destroy();
      this._chart = null;

      this.$emit('chart:destroy');
    }
  },

  render (h) {
    return h('div', [
      h('canvas', {
        ref: 'canvas',
        style: 'width:100%; height:100%;'
      })
    ]);
  },

  mounted () {
    this.render();
  },

  beforeDestroy () {
    if (this._chart) {
      this._chart.destroy();
    }
  },
};
</script>
