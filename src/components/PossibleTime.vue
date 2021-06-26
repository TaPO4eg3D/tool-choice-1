<template>
  <div class="possible-time">
    <h1>Возможные сроки выполнения</h1>

    <v-btn
      v-if="!showProduct"
      class="product-show-btn"
      color="primary"
      @click="showProduct = true"
    >Показать все возможные комбинации</v-btn>

    <v-btn
      v-if="showProduct"
      class="product-show-btn"
      color="error"
      @click="showProduct = false"
    >Скрыть все возможные комбинации</v-btn>

    <div class="combination-tables" v-if="showProduct">
      <v-simple-table>
        <template v-slot:default>
          <thead>
            <tr>
              <th class="text-left" v-for="(action, i) in actions" :key="i">
                T_{{ action.codeName }}
              </th>
              <th>T</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(combinations, i) in actionsProduct" :key="i">
              <td v-for="(combination, j) in combinations" :key="j">{{ combination }}</td>
              <td>{{ getBuildingTime(combinations) }}</td>
            </tr>
          </tbody>
        </template>
      </v-simple-table>

      <v-simple-table class="product-result" v-if="showProduct">
        <template v-slot:default>
          <thead>
            <tr>
              <th class="text-center" v-for="(value, key) in actionsProductStats" :key="key">
                T_{{ key }}
              </th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td class="text-center" v-for="(value, key) in actionsProductStats" :key="key">
                <b>{{ value }}</b>
              </td>
            </tr>
            <tr>
              <td class="text-center" v-for="(value, key) in actionsProductStats" :key="key">
                <b>{{ (value / actionsProduct.length).toFixed(2) }} %</b>
              </td>
            </tr>
          </tbody>
        </template>
      </v-simple-table>
    </div>


  </div>
</template>

<script>
export default {
  name: 'possible-time',
  props: ['actions', 'actionsProduct', 'actionsProductStats'],
  data() {
    return {
      showProduct: false,
    }
  },
  methods: {
    getBuildingTime(actionValues) {
      return Math.max(actionValues[0] + actionValues[3], actionValues[2], actionValues[1] + actionValues[4]);
    },
  }
}
</script>

<style lang="scss">
.possible-time {
  margin-top: 40px;
}

.combination-tables {
  display: flex;
}

.product-show-btn {
  margin: 10px 0;
}

.product-result {
  margin-left: 60px;
}
</style>
