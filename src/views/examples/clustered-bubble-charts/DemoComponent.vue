<template>
  <div class="fill">
    <input v-model.number="forceStrength" />
    <svg>
      <circle v-for="(node, index) in nodePositions"
              :key="index"
              :cx="node.x"
              :cy="node.y"
              :fill="fillColor(node.value)"
              :r="node.radius"></circle>
    </svg>
  </div>
</template>

<script>
import * as d3 from 'd3'
import chroma from 'chroma-js'
// Nice looking colors - no reason to buck the trend
// @v4 scales now have a flattened naming scheme

window.d3 = d3
const fillColor = d3
  .scaleQuantile()
  .domain(['low', 'medium', 'high'])
  .range(chroma.scale(chroma.brewer.RdYlBu.reverse()).colors(3))

export default {
  name: 'ClusteredBubbleCharts',
  data() {
    return {
      fillColor,
      center:        { x: 0, y: 0 },
      forceStrength: 0.03,
      rawData:       null,

      /** @type {d3.Simulation} */
      simulation:  null,
      yearCenters: {
        2008: {
          x: -200,
          y: 0
        },
        2009: {
          x: 0,
          y: 0
        },
        2010: {
          x: 200,
          y: 0
        },
        low: {
          x: -500,
          y: 333
        },
        medium: {
          x: 0,
          y: 0
        },
        high: {
          x: 300,
          y: -500
        }
      },
      showYears:     false,
      orgNames:      [],
      nodePositions: []
    }
  },
  computed: {
    nodes() {
      if (!this.rawData) return
      // Use the max total_amount in the data as the max in the scale's domain
      // note we have to ensure the total_amount is a number.
      const maxAmount = d3.max(this.rawData, d => {
        return +d.total_amount
      })

      const domainVal = [0, maxAmount]

      fillColor.domain(domainVal)

      // Sizes bubbles based on area.
      // @v4: new flattened scale names.
      const radiusScale = d3
        .scalePow()
        .exponent(0.6)
        .range([5, 85])
        .domain(domainVal)

      // Use map() to convert raw data into node data.
      // Checkout http://learnjsdata.com/ for more on
      // working with data.
      this.orgNames = []
      const myNodes = this.rawData.map(d => {
        if (!this.orgNames.includes(d.group)) {
          this.orgNames.push(d.group)
        }
        return {
          id:     d.id,
          radius: radiusScale(+d.total_amount),
          value:  +d.total_amount,
          name:   d.grant_title,
          org:    d.organization,
          group:  d.group,
          year:   d.start_year,
          x:      Math.random() * 900,
          y:      Math.random() * 800
        }
      })

      // sort them to prevent occlusion of smaller nodes.
      myNodes.sort((a, b) => {
        return b.value - a.value
      })

      return myNodes
    }
  },
  methods: {
    charge(d) {
      return -this.forceStrength * Math.pow(d.radius, 2)
    },
    ticked() {
      this.nodePositions = Object.freeze(
        this.simulation.nodes().map(v => ({
          x:      v.x,
          y:      v.y,
          radius: v.radius,
          value:  v.value
        }))
      )
    }
  },
  mounted() {
    // d3.json('/static/demo_data/quantifiable/populations.json').then(res => {
    d3.csv('/static/demo_data/gates.csv').then(res => {
      this.rawData = res
    })
  },
  watch: {
    nodes() {
      this.simulation = d3
        .forceSimulation(this.nodes)
        .velocityDecay(0.3)
        .force(
          'x',
          d3
            .forceX()
            .strength(this.forceStrength)
            .x(this.center.x)
        )
        .force(
          'y',
          d3
            .forceY()
            .strength(this.forceStrength)
            .y(this.center.y)
        )
        .force('center', d3.forceManyBody().strength(this.charge))
        .on('tick', this.ticked)

      this.$nextTick(() => {
        d3.drag(this.nodePositions)(d3.selectAll('circle'))
      })
    },
    showYears(val) {
      const r = Math.random() > 0.5
      if (this.simulation) {
        this.simulation.force(
          'x',
          d3
            .forceX()
            .strength(this.forceStrength)
            .x(d => {
              return val
                ? this.yearCenters[r ? d.group : d.year].x
                : this.center.x
            })
        )

        this.simulation.force(
          'y',
          d3
            .forceY()
            .strength(this.forceStrength)
            .y(d => {
              return val
                ? this.yearCenters[r ? d.group : d.year].y
                : this.center.y
            })
        )

        this.simulation.alpha(1.5).restart()
      }
    }
  }
}
</script>

<style scoped>
.fill {
  height: 100%;
  transform: translateZ(0);
}

circle {
  stroke: #000;
  stroke-width: 2px;
}
</style>
