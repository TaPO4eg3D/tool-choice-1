<template>
  <div class="service-table">
    <h1>Виды работ Завода</h1>
    <div>
      <v-simple-table
        fixed-header
        height="300px"
      >
        <template v-slot:default>
          <thead>
            <tr>
              <th class="text-left name-col">
                Наименование работ
              </th>
              <th class="text-left">
                Время выполнения (года)
              </th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(action, i) in actions" :key="i">
              <td class="text-left">{{ action.name }} ({{ action.codeName }})</td>
              <div class="execution-changer">
                <v-btn
                  class="mx-2 add-button"
                  fab
                  dark
                  small
                  color="indigo"
                  @click="addNewActionValue(i)"
                >
                  <v-icon dark>
                    mdi-plus
                  </v-icon>
                </v-btn>
                <v-text-field
                  v-for="(value, vi) in action.value" :key="vi"
                  @input="onValueChange(i, vi, +$event)"
                  @keydown="onKeyPress($event, i, vi)"
                  :value="value"
                  class="value-input"
                  type="number"
                  min="1"
                >
                </v-text-field>
              </div>
            </tr>
          </tbody>
        </template>
      </v-simple-table>
    </div>
    <possible-time
      :actions="actions"
      :actionsProduct="actionsProduct"
      :actionsProductStats="actionsProductStats"
    ></possible-time>
    <div class="time-profit">
      <h1>Прибыль завода</h1>
      <div class="money-reserve">
        <span>Дополнительный резер</span>
        <v-text-field v-model="moneyReserve.value" type="number"></v-text-field>
        <span>ускорит строительство на: </span>
        <v-text-field v-model="moneyReserve.speed" :min="1" :max="maxReserveValue" type="number"></v-text-field>
      </div>
      <v-simple-table>
        <template v-slot:default>
          <thead>
            <tr>
              <th v-for="(value, key) in timeProfit" :key="key">
                T = {{ key }}
              </th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td v-for="(value, key) in timeProfit" :key="key">
                <v-text-field 
                  type="number"
                  v-model="timeProfit[key]"
                  ></v-text-field>
              </td>
            </tr>
            <tr>
              <td v-for="(value, key) in timeProfit" :key="key">
                <div v-if="timeProfit[key - moneyReserve.speed] !== undefined">
                  {{ timeProfit[key - moneyReserve.speed] - moneyReserve.value }}
                </div>
              </td>
            </tr>
          </tbody>
        </template>
      </v-simple-table>
    </div>
    <div class="max-possibility">
      <h1>Оценки</h1>
      <div class="cards">
        <v-card
          outlined
        >
          <v-list-item three-line>
            <v-list-item-content>
              <div class="text-overline mb-4">
                <v-chip v-if="isMostPossibilityRelevant"
                  color="green"
                >Целесообразно</v-chip>
                <v-chip v-else
                  color="red"
                >Нецелесообразно</v-chip>
              </div>
              <v-list-item-title class="text-h5 mb-1">
                Максимальное правдоподобие
              </v-list-item-title>
              <v-list-item-subtitle class="mt-3">
                <div>
                  M(0, {{ mostPossibleAction.key }}) = {{ timeProfit[mostPossibleAction.key] }}
                </div>
                <div class="mt-2">
                  M(1, {{ mostPossibleAction.key }}) = {{ timeProfit[mostPossibleAction.key - moneyReserve.speed] - moneyReserve.value }}
                </div>
              </v-list-item-subtitle>
            </v-list-item-content>
          </v-list-item>
        </v-card>
        <v-card
          outlined
        >
          <v-list-item three-line>
            <v-list-item-content>
              <div class="text-overline mb-4">
                <v-chip v-if="bayasWithReserve > bayas"
                  color="green"
                >Целесообразно</v-chip>
                <v-chip v-else
                  color="red"
                >Нецелесообразно</v-chip>
              </div>
              <v-list-item-title class="text-h5 mb-1">
                Принцип Байеса
              </v-list-item-title>
              <v-list-item-subtitle class="mt-3">
                <div>
                  M(0, {{ mostPossibleAction.key }}) = 
                  <span v-for="(value, key, i) in actionsProductStats" :key="key">
                    {{ timeProfit[key] }} * {{ value }}/{{ actionsProductPower }}

                    <span v-if="i !== Object.keys(actionsProductStats).length - 1">+</span>
                  </span>
                  = {{ bayas.toFixed(2) }}
                </div>
                <div class="mt-2">
                  M(1, {{ mostPossibleAction.key }}) = 
                  <span v-for="(value, key, i) in actionsProductStats" :key="key">
                    {{ timeProfit[key - moneyReserve.speed] - moneyReserve.value }} * {{ value }}/{{ actionsProductPower }}

                    <span v-if="i !== Object.keys(actionsProductStats).length - 1">+</span>
                  </span>
                  = {{ bayasWithReserve.toFixed(2) }}
                </div>
              </v-list-item-subtitle>
            </v-list-item-content>
          </v-list-item>
        </v-card>
        <v-card
          outlined
        >
          <v-list-item three-line>
            <v-list-item-content>
              <div class="text-overline mb-4">
                <v-chip
                  color="green"
                >Целесообразно</v-chip>
              </div>
              <v-list-item-title class="text-h5 mb-1">
                Гарантированная оценка
              </v-list-item-title>
              <v-list-item-subtitle class="mt-3">
                <div>
                  T = {{ worstPossibleAction.key }}
                </div>
              </v-list-item-subtitle>
            </v-list-item-content>
          </v-list-item>
        </v-card>
      </div>
    </div>
    <div></div>
    <div></div>
  </div>
</template>

<script>
import PossibleTime from '@/components/PossibleTime.vue';

import { CartesianProduct } from 'js-combinatorics';

export default {
  name: 'service-table',
  data() {
    return {
      actions: [
        {
          name: 'Строительство корпусов',
          codeName: 'A',
          value: [2],
        },
        {
          name: 'Разработка модели изделия',
          codeName: 'B',
          value: [2],
        },
        {
          name: 'Наем и обучение рабочей силы',
          codeName: 'C',
          value: [2, 3, 4],
        },
        {
          name: 'Монтаж оборудования',
          codeName: 'D',
          value: [2, 3, 4],
        },
        {
          name: 'Отладка модели изделия',
          codeName: 'E',
          value: [2, 3, 4],
        },
      ],
      moneyReserve: {
        value: 20,
        speed: 2,
      },
      timeProfit: {},
    };
  },
  components: {
    PossibleTime,
  },
  mounted() {
    this.buildTimeProfitTable();
  },
  watch: {
    actionsProduct: {
      handler() {
        this.buildTimeProfitTable();
      },
    },
    'moneyReserve.speed': {
      handler() {
        this.buildTimeProfitTable();
      }
    },
  },
  computed: {
    maxReserveValue() {
      return +Object.keys(this.actionsProductStats)[0];
    },
    actionsProduct() {
      const data = new CartesianProduct(...this.actions.map(action => [...action.value]));
      return [...data];
    },
    actionsProductPower() {
      return (this.actionsProduct || []).length;
    },
    actionsProductStats() {
      const stats = {};

      this.actionsProduct.forEach((actionProduct) => {
        const buildingTime = this.getBuildingTime(actionProduct);

        if (stats[buildingTime] === undefined) {
          stats[buildingTime] = 1;
        } else {
          stats[buildingTime] += 1;
        }
      });

      return stats;
    },
    mostPossibleAction() {
      const action = { key: 0, value: 0};

      Object.entries(this.actionsProductStats).forEach(([key, value]) => {
        const possibility = value / this.actionsProductPower;

        if (possibility > action.value) {
          action.key = key;
          action.value = possibility;
        }
      });

      return action;
    },
    worstPossibleAction() {
      const action = { key: 0, value: 9999};

      Object.entries(this.actionsProductStats).forEach(([key, value]) => {
        const possibility = value / this.actionsProductPower;

        if (possibility < action.value) {
          action.key = key;
          action.value = possibility;
        }
      });

      return action;
    },
    isMostPossibilityRelevant() {
      const profit = this.timeProfit[this.mostPossibleAction.key];
      const profitWithReserve = this.timeProfit[this.mostPossibleAction.key - this.moneyReserve.speed] - this.moneyReserve.value;

      return profitWithReserve > profit;
    },
    bayas() {
      let bayasValue = 0;

      Object.entries(this.actionsProductStats).forEach(([key, value]) => {
        bayasValue += +this.timeProfit[key] * (value / this.actionsProductPower);
      });
      
      return bayasValue;
    },
    bayasWithReserve() {
      let bayasValue = 0;

      Object.entries(this.actionsProductStats).forEach(([key, value]) => {
        bayasValue += (this.timeProfit[key - this.moneyReserve.speed] ) * (value / this.actionsProductPower);
      });
      
      return bayasValue;
    },
  },
  methods: {
    addNewActionValue(actionIndex) {
      this.actions[actionIndex].value.push(0);
    },
    onValueChange(actionIndex, valueIndex, value) {
      this.actions[actionIndex].value.splice(valueIndex, 1, value);
    },
    onKeyPress({ keyCode }, actionIndex, valueIndex) {
      // Delete button
      if (keyCode === 46) {
        this.actions[actionIndex].value.splice(valueIndex, 1)
      }
    },
    getBuildingTime(actionProduct) {
      return Math.max(actionProduct[0] + actionProduct[3], actionProduct[2], actionProduct[1] + actionProduct[4]);
    },
    buildTimeProfitTable() {
      let minT = 9999;
      let maxT = 0;

      this.moneyReserve.speed = +this.moneyReserve.speed;

      Object.keys(this.actionsProductStats).forEach((value) => {
        if (value > maxT) {
          maxT = value;
        }

        if (value < minT) {
          minT = value;
        }
      });

      let baseProfit = 1000;
      let profitModifier = Math.floor(baseProfit / (+maxT + this.moneyReserve.speed));

      const timeProfit = {}
      for (let i = minT - this.moneyReserve.speed; i <= +maxT + this.moneyReserve.speed; i++) {
        timeProfit[i] = baseProfit - i * profitModifier;
      }

      this.timeProfit = timeProfit;
    },
  },
}
</script>

<style lang="scss">
.service-table {
  h1 {
    margin-bottom: 10px;
  }

  .name-col {
    width: 300px;
  }

  margin-bottom: 40px;
}

.execution-changer {
  display:flex;

  align-items: center;

  .add-button {
    margin-right: 20px !important;
  }

  .value-input {
    max-width: 30px;
    margin-right: 8px;
  }
}

.time-profit {
  margin-top: 30px;
}

.money-reserve {
  display: flex;
  
  align-items: center;

  .v-input {
    margin: 0px 8px;
    max-width: 50px;
  }
}

.max-possibility {
  margin-top: 40px;
}

.cards {
  display: flex;

  .v-card {
    margin-right: 20px;
  }
}
</style>
