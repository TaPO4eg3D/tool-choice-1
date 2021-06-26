<template>
  <v-app>
    <v-main>
      <service-table></service-table>
    </v-main>
  </v-app>
</template>

<script>
import ServiceTable from '@/components/ServiceTable.vue'

export default {
  components: {
    ServiceTable,
  }
}
</script>

<style lang="scss">
html {
  padding: 20px;
}
</style>
