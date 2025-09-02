<template>
  <div class="currency-list">
    <h2>Currency Rates</h2>
    <input
      type="text"
      v-model="filterText"
      placeholder="Filter by currency code, name, or rate"
      class="filter-input"
    />

    <table>
      <thead>
        <tr>
          <th class="currency-col">Currency</th>
          <th class="rate-col">Online Sell Rate</th>
        </tr>
      </thead>
    </table>

    <!-- Scrollable tbody container -->
    <div class="table-scroll">
      <table>
        <tbody>
          <tr v-for="currency in filteredCurrencies" :key="currency.short_code">
            <td class="currency-cell">
              <img
                :src="flagUrl(currency.short_code)"
                alt="flag"
                class="flag"
              />
              <span>{{ currency.name }} ({{ currency.short_code }})</span>
            </td>
            <td class="rate-cell">
              {{ formatRate(currencyRate(currency.short_code)) }}
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <div v-if="error" class="error">{{ error }}</div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import axios from "axios";
import "./CurrencyList.css";

const apiKey = "1SMxQumaAcC996SUXGsnlBPW44t2RR82";
const currencies = ref([]);
const rates = ref({});
const filterText = ref("");
const error = ref(null);
const baseCurrency = "USD";

// Fetch currencies list
async function fetchCurrencies() {
  try {
    const response = await axios.get(
      `https://api.currencybeacon.com/v1/currencies?api_key=${apiKey}`
    );
    currencies.value = response.data.response;
  } catch (e) {
    error.value = "Failed to load currencies";
  }
}

// Fetch latest rates relative to USD
async function fetchRates() {
  try {
    const response = await axios.get(
      `https://api.currencybeacon.com/v1/latest?api_key=${apiKey}`
    );
    rates.value = response.data.rates;
  } catch (e) {
    error.value = "Failed to load rates";
  }
}

// Computed filtered currencies based on filter input
const filteredCurrencies = computed(() => {
  if (!filterText.value.trim()) return currencies.value;
  const term = filterText.value.toLowerCase();

  return currencies.value.filter((c) => {
    const codeMatch = c.short_code.toLowerCase().includes(term);
    const nameMatch = c.name.toLowerCase().includes(term);
    const rate = currencyRate(c.short_code);
    const rateMatch = rate && rate.toString().includes(term);
    return codeMatch || nameMatch || rateMatch;
  });
});

// Rate formatter
function formatRate(rate) {
  return rate ? rate.toFixed(4) : "-";
}

// Helper to get rate for currency code, fallback to null
function currencyRate(code) {
  // For USD itself rate is 1
  if (code === baseCurrency) return 1;
  return rates.value[code] ?? null;
}

// Returns flag URL (same logic as top section)
function flagUrl(code) {
  const map = {
    EUR: "eu",
    XOF: "sn",
    XAF: "cm",
    XPF: "fr",
    USD: "us",
    AUD: "au",
    INR: "in",
    CAD: "ca",
    GBP: "gb",
    NZD: "nz",
  };
  const country = map[code] || code.slice(0, 2).toLowerCase();
  return `https://flagcdn.com/w40/${country}.png`;
}

onMounted(() => {
  fetchCurrencies();
  fetchRates();
});
</script>
