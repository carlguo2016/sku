<template>
  <div id="app">
    <div class="container">
      <specification />
    </div>
  </div>
</template>

<script>
import Specification from '@/components/specification'

export default {
  components: {
    Specification
  }
}
</script>

<style lang="scss">
.container {
  width: 1000px;
  margin: auto;
}
</style>
