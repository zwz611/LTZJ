<template>
  <div class="chart-container">
    <chart height="100%" width="100%" />
  </div>
</template>
<script>
import Chart from '@/components/Charts/memberChart'
export default {
  name: 'MumChart',
  components: { Chart }
}
</script>

<style>

  .chart-container{
    position: relative;
    width: 100%;
    height: calc(100vh - 84px);
  }

</style>
