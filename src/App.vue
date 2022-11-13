<template>
  <div class="container mx-auto flex flex-col items-center bg-gray-100 p-4">
    <div class="container">
      <div class="w-full my-4"></div>
      <search-crypto-coin @add-ticker="add" :error="error" />
      <tickers-list
        v-if="tickers.length"
        @select-ticker="select"
        @delete-ticker="deleteTicker"
        :tickers="tickers"
        :selectedTicker="selectedTicker"
      />
      <sell-diagram
        v-if="selectedTicker"
        :selectedTickerName="selectedTicker.name"
        :graph="graph"
        @remove-select-ticker="removeSelect"
      />
    </div>
  </div>
</template>

<script>
import SearchCryptoCoin from "./components/SearchCryptoCoin.vue";
import SellDiagram from "./components/SellDiagram.vue";
import TickersList from "./components/TickersList.vue";
import { subscribeToTicker, unsubscribeFromTicker } from "./utils/api";
export default {
  components: { SearchCryptoCoin, TickersList, SellDiagram },
  name: "App",

  data() {
    return {
      tickers: [],
      selectedTicker: null,
      error: "",
      graph: [],
    };
  },
  created() {
    const tickerData = localStorage.getItem("crypto-list");
    if (tickerData) {
      const tickerList = JSON.parse(tickerData);
      this.tickers = tickerList;
      this.tickers.forEach((t) => {
        subscribeToTicker(t.name, (newPrice) =>
          this.updateTicker(t.name, newPrice)
        );
      });
    }
  },
  watch: {
    tickers() {
      localStorage.setItem("crypto-list", JSON.stringify(this.tickers));
    },
    selectedTicker() {
      this.graph = [];
    },
  },
  methods: {
    updateTicker(tickerName, price) {
      this.tickers
        .filter((t) => t.name === tickerName)
        .forEach((t) => {
          if (t === this.selectedTicker) {
            this.graph.push(price);
          }
          t.price = price;
        });
    },
    add(ticker) {
      const currentTicker = {
        id: this.tickers.length + 1,
        name: ticker.toUpperCase(),
        price: "-",
      };
      if (this.check(currentTicker.name)) {
        this.tickers = [...this.tickers, currentTicker];
        subscribeToTicker(currentTicker.name, (newPrice) =>
          this.updateTicker(currentTicker.name, newPrice)
        );
      }
    },
    deleteTicker(ticker) {
      this.tickers = this.tickers.filter((t) => t.id !== ticker.id);
      if (this.selectedTicker == ticker) this.selectedTicker = null;
      unsubscribeFromTicker(ticker.name);
    },
    check(tickerName) {
      if (this.tickers.find((t) => t.name == tickerName)) {
        this.error = "Вы уже выбрали такую криптовалюту";
        return false;
      }
      this.error = "";
      return true;
    },
    select(ticker) {
      this.selectedTicker = ticker;
    },
    removeSelect(sel) {
      this.selectedTicker = sel;
    },
  },
};
</script>
<style>
</style>