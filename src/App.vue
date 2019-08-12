<template>
  <div id="app">
    <div class="container">
      <specification :spec="spec" :sku="sku" @change-spec="handleChangeSpec" @change-sku="handleChangeSku" />
    </div>
  </div>
</template>

<script>
import Specification from '@/components/specification'

export default {
  data () {
    return {
      spec: [],
      sku: []
    }
  },

  components: {
    Specification
  },

  methods: {
    handleChangeSpec (spec) {
      this.spec = spec
    },

    handleChangeSku (sku) {
      this.sku = sku
    }
  }
}
</script>

<style lang="scss">
.container {
  width: 1000px;
  margin: auto;
}
</style>
