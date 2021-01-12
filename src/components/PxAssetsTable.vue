<template>
  <table class="container">
    <thead>
      <tr class="bg-gray-100 border-b-2 border-gray-400">
        <th></th>
        <th>
          <span>Ranking</span>
        </th>
        <th>Nombre</th>
        <th>Precio</th>
        <th>Cap. de Mercado</th>
        <th>Variación 24hs</th>
        <td class="hidden sm:block">
          <input
            type="text"
            id="filter"
            class="bg-gray-100 focus:outline-none border-b border-gray-400 py-2 px-4 block"
            placeholder="Buscar.."
            v-model="filter"
          />
        </td>
      </tr>
    </thead>
    <tbody>
      <tr
        v-for="a in filteredAssets"
        :key="a.id"
        class="border-b border-gray-200 hover:bg-gray-100 hover:bg-orange-100"
      >
        <td>
          <img
            :src="
              `https://static.coincap.io/assets/icons/${a.symbol.toLowerCase()}@2x.png`
            "
            :alt="a.name"
          />
        </td>
        <td>
          <b># {{ a.rank }}</b>
        </td>
        <td>
          <router-link
            :to="`/coin/${a.id}`"
            class="hover:underline text-green-600"
            >{{ a.name }}</router-link
          >
        </td>
        <td>{{ parseFloat(a.priceUsd).toFixed(2) + "$" }}</td>
        <td>{{ parseFloat(a.marketCapUsd).toFixed(2) + "$" }}</td>
        <td
          :class="{
            red: a.changePercent24Hr < 0,
            green: a.changePercent24Hr >= 0,
          }"
        >
          {{ parseFloat(a.changePercent24Hr).toFixed(2) + "%" }}
        </td>
        <td class="hidden sm:block"></td>
      </tr>
    </tbody>
  </table>
</template>

<script>
export default {
  name: "PxAssetsTable",
  data() {
    return {
      filter: "",
    };
  },

  props: {
    assets: {
      type: Array,
      default: () => [],
    },
  },

  computed: {
    filteredAssets() {
      if (!this.assets) {
        return this.assets;
      }
      return this.assets.filter(
        (x) =>
          x.symbol.toLowerCase().includes(this.filter.toLowerCase()) ||
          x.name.toLowerCase().includes(this.filter.toLowerCase())
      );
    },
  },
};
</script>

<style scoped>
.red {
  color: red;
}
.green {
  color: green;
}

img {
  max-width: 70px;
}
.up::before {
  content: "👆";
}

.down::before {
  content: "👇";
}

td {
  padding: 20px 0px;
  font-size: 0.6rem;
  text-align: center;
}

th {
  padding: 5px;
  font-size: 0.6rem;
}

@media (min-width: 640px) {
  td,
  th {
    padding: 20px;
    font-size: 1rem;
  }

  th {
    padding: 12px;
  }
}
</style>
