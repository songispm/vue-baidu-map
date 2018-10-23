<template>
<span>
  <slot></slot>
</span>  
</template>

<script>
import MarkerClusterer from 'bmaplib.markerclusterer'
import {createSize} from '../base/factory.js'
import commonMixin from '../base/mixins/common.js'

export default {
  name: 'bml-marker-cluseter',
  mixins: [commonMixin('markerClusterer')],
  props: {
    gridSize: {
      type: Object
    },
    maxZoom: {
      type: Number
    },
    minClusterSize: {
      type: Number
    },
    styles: {
      type: Array,
      default () {
        return []
      }
    },
    averageCenter: {
      type: Boolean,
      default: false
    }
  },
  watch: {
    gridSize: {
      handler (val) {
        const {BMap, originInstance} = this
        originInstance.setGridSize(BMap, val.map)
      },
      deep: true
    },
    maxZoom (val) {
      const {originInstance} = this
      originInstance.setMaxZoom(val)
    },
    minClusterSize: {
      handler (val) {
        const {BMap, originInstance} = this
        originInstance.setMinClusterSize(createSize(BMap, val))
      },
      deep: true
    },
    styles: {
      handler (val) {
        const {BMap, originInstance} = this
        const obj = JSON.parse(JSON.stringify(val)).map(item => {
          item.size = item.size && createSize(BMap, item.size)
          return item
        })
        originInstance.setStyles(obj)
      },
      deep: true
    },
    averageCenter (val) {
      this.reload()
    }
  },
  methods: {
    load () {
      const {BMap, map, gridSize, minClusterSize, maxZoom, styles, averageCenter} = this
      this.initNum++
      this.$nextTick(() => {
        this.nextTickNum++
        if (this.nextTickNum !== this.initNum) {
          return
        }
        this.initNum = 0
        this.nextTickNum = 0
        this.originInstance = new MarkerClusterer(map, {
          gridSize: gridSize && createSize(BMap, gridSize),
          maxZoom,
          minClusterSize: minClusterSize || 2,
          styles: styles.map(item => {
            item.size = createSize(BMap, item.size)
            return item
          }),
          isAverageCenter: averageCenter
        })
        const markers = this.$children.map(inst => inst.originInstance).filter(marker => marker instanceof BMap.Marker)
        this.originInstance.addMarkers(markers)
      })
    }
  },
  beforeCreate () {
    this.preventChildrenRender = true
  }
}
</script>
