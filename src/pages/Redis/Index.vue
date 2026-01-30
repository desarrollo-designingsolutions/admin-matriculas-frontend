<script setup lang="ts">
import CacheHitRateChart from "@/pages/Redis/Components/CacheHitRateChart.vue";
import ListCacheKeys from "@/pages/Redis/Components/ListCacheKeys.vue";
import RequestVolumeChart from "@/pages/Redis/Components/RequestVolumeChart.vue";
import ResponseTimeChart from "@/pages/Redis/Components/ResponseTimeChart.vue";
definePage({
  name: "Redis-Index",
});


</script>

<template>
  <div>
    <VRow>
      <VCol cols="12">
        <ListCacheKeys />
      </VCol>
      <VCol cols="12" md="6">
        <RequestVolumeChart />
      </VCol>
      <VCol cols="12" md="6">
        <ResponseTimeChart />
      </VCol>
      <VCol cols="12" md="6">
        <CacheHitRateChart />
      </VCol>
    </VRow>
  </div>
</template>
