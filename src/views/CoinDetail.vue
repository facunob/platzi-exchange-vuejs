<template>
  <div class="flex-col">
    <div class="flex justify-center">
      <bounce-loader :loading="isLoading" :size="100" :color="'#68d391'" />
    </div>

    <template v-if="!isLoading">
      <div class="flex flex-col sm:flex-row justify-around items-center">
        <div class="flex flex-col items-center">
          <img
            :src="
              `https://static.coincap.io/assets/icons/${asset.symbol.toLowerCase()}@2x.png`
            "
            :alt="asset.name"
            class="w-20 h-20 mr-5"
          />
          <h1 class="text-5xl">
            {{ asset.name }}
            <small class="sm:mr-2 text-gray-500">{{ asset.symbol }}</small>
          </h1>
        </div>

        <div class="my-10 flex flex-col">
          <ul>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Ranking</b>
              <span>#{{ asset.rank }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio actual</b>
              <span>{{ parseFloat(asset.priceUsd).toFixed(2) }}$</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio más bajo</b>
              <span>{{ min }}$</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio más alto</b>
              <span>{{ max }}$</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio Promedio</b>
              <span>{{ avg }}$</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Variación 24hs</b>
              <span>{{ parseFloat(asset.changePercent24Hr).toFixed(2) }}%</span>
            </li>
          </ul>
        </div>

        <div
          v-if="usd"
          class="my-10 sm:mt-0 flex flex-col justify-center text-center"
        >
          <button
            class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded"
            v-on:click="usdBool"
          >
            De USD a {{ asset.symbol }}
          </button>

          <div class="flex flex-row my-5">
            <label class="w-full" for="convertValue">
              <input
                v-model="value"
                id="convertValue"
                type="number"
                class="text-center bg-white focus:outline-none focus:shadow-outline border border-gray-300 rounded-lg py-2 px-4 block w-full appearance-none leading-normal"
              />
            </label>
          </div>

          <span class="text-xl">{{
            parseFloat(value / asset.priceUsd).toFixed(2) + " " + asset.symbol
          }}</span>
        </div>

        <div
          v-else
          class="my-10 sm:mt-0 flex flex-col justify-center text-center"
        >
          <button
            class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded"
            v-on:click="usdBool"
          >
            De {{ asset.symbol }} a USD
          </button>

          <div class="flex flex-row my-5">
            <label class="w-full" for="convertValue">
              <input
                v-model="value"
                id="convertValue"
                type="number"
                class="text-center bg-white focus:outline-none focus:shadow-outline border border-gray-300 rounded-lg py-2 px-4 block w-full appearance-none leading-normal"
              />
            </label>
          </div>

          <span class="text-xl">{{
            parseFloat(asset.priceUsd * value).toFixed(2) + " USD"
          }}</span>
        </div>
      </div>

      <line-chart
        class="my-10"
        :colors="['orange']"
        :min="min"
        :max="max"
        :data="history.map((x) => [x.date, parseFloat(x.priceUsd).toFixed(2)])"
      />

      <h3 class="text-xl my-10">Mejores Ofertas de Cambio</h3>
      <table>
        <tr
          v-for="m in markets"
          :key="`${m.exchangeId}-${m.priceUsd}`"
          class="border-b"
        >
          <td>
            <b>{{ m.exchangeId }}</b>
          </td>
          <td>{{ parseFloat(m.priceUsd).toFixed(2) }}$</td>
          <td>{{ m.baseSymbol }} / {{ m.quoteSymbol }}</td>
          <td>
            <px-button v-if="!m.url" v-on:custom-click="getWebsite(m)">
              <slot>Obtener Link</slot>
            </px-button>
            <a v-else>{{ m.url }}</a>
          </td>
        </tr>
      </table>
    </template>
  </div>
</template>

<script>
import api from "@/api";
import PxButton from "@/components/PxButton";

export default {
  name: "CoinDetail",
  components: { PxButton },
  data() {
    return {
      markets: [],
      asset: {},
      history: [],
      isLoading: false,
      value: 0,
      usd: true,
    };
  },

  computed: {
    min() {
      let min = this.history[0].priceUsd;
      for (let i = 1; i < this.history.length; i++) {
        if (this.history[i].priceUsd < min) {
          min = this.history[i].priceUsd;
        }
      }
      min = parseFloat(min);
      min = min.toFixed(2);
      return min;
    },
    max() {
      let max = this.history[0].priceUsd;
      for (let i = 1; i < this.history.length; i++) {
        if (this.history[i].priceUsd > max) {
          max = this.history[i].priceUsd;
        }
      }
      max = parseFloat(max);
      max = max.toFixed(2);
      return max;
    },
    avg() {
      let avg = 0;
      for (let i = 0; i < this.history.length; i++) {
        avg = avg + parseFloat(this.history[i].priceUsd);
      }
      avg = avg / this.history.length;
      return avg.toFixed(2);
    },
  },

  watch: {
    $route() {
      this.getCoin();
    },
  },

  created() {
    this.getCoin();
  },
  methods: {
    usdBool() {
      this.usd = !this.usd;
    },
    getWebsite(exchange) {
      return api
        .getExchange(exchange.exchangeId)
        .then((res) => this.$set(exchange, "url", res.exchangeUrl));
    },
    getCoin() {
      this.isLoading = true;
      const id = this.$route.params.id;
      Promise.all([
        api.getAsset(id),
        api.getAssetHistory(id),
        api.getMarkets(id),
      ])
        .then(([asset, history, markets]) => {
          this.asset = asset;
          this.history = history;
          this.markets = markets;
        })
        .finally(() => (this.isLoading = false));
    },
  },
};
</script>

<style scoped>
td {
  padding: 10px;
  text-align: center;
}
</style>
