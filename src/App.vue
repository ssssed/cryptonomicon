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
        this.subscribeToUpdate(t.name);
      });
    }
  },
  watch: {
    tickers() {
      localStorage.setItem("crypto-list", JSON.stringify(this.tickers));
    },
    select() {
      this.graph = [];
    },
  },
  methods: {
    add(ticker) {
      const currentTicker = {
        id: this.tickers.length + 1,
        name: ticker,
        price: "-",
      };
      if (this.check(currentTicker.name)) {
        this.tickers.push(currentTicker);
        this.subscribeToUpdate(currentTicker.name);
      }
    },
    deleteTicker(ticker) {
      this.tickers = this.tickers.filter((t) => t.id !== ticker.id);
      if (this.selectedTicker == ticker) this.selectedTicker = null;
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
    subscribeToUpdate(tickerName) {
      setInterval(async () => {
        const f = await fetch(
          `https://min-api.cryptocompare.com/data/price?fsym=${tickerName}&tsyms=USD&api_key=7a7ee3375c78d35cc3def34b98e347b7864691d1bda4dfe7b2eb98566df823d9`
        );
        const data = await f.json();
        this.tickers.find((t) => t.name === tickerName).price =
          data.USD > 1 ? data.USD.toFixed(2) : data.USD.toPrecision(2);
        if (this.selectedTicker?.name === tickerName) {
          this.graph.push(data.USD);
        }
      }, 5000);
    },
  },
};
</script>
<style>
</style>