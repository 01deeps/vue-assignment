<script setup>
import { ref, onMounted, watch } from "vue";
import axios from "axios";
import "./CurrencyConverter.css";
import CustomDropdown from "./CustomDropdown.vue";

const apiKey = "1SMxQumaAcC996SUXGsnlBPW44t2RR82";
const currencies = ref([]);
const fromCurrency = ref("AUD");
const toCurrency = ref("USD");
const amount = ref(1000);
const result = ref(null);
const error = ref(null);

// Flag URL
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

async function fetchCurrencies() {
  try {
    const response = await axios.get(
      `https://api.currencybeacon.com/v1/currencies?api_key=${apiKey}`
    );
    currencies.value = response.data.response;
  } catch {
    error.value = "Failed to load currencies";
  }
}

async function convert() {
  error.value = null;
  result.value = null;
  if (!amount.value || amount.value <= 0) {
    error.value = "Please enter a valid amount";
    return;
  }
  try {
    const url = `https://api.currencybeacon.com/v1/convert?api_key=${apiKey}&from=${fromCurrency.value}&to=${toCurrency.value}&amount=${amount.value}`;
    const response = await axios.get(url);
    if ("converted_amount" in response.data) {
      result.value = new Intl.NumberFormat().format(
        response.data.converted_amount
      );
    } else if (
      "response" in response.data &&
      "value" in response.data.response
    ) {
      result.value = new Intl.NumberFormat().format(
        response.data.response.value
      );
    } else {
      error.value = "Conversion failed";
    }
  } catch {
    error.value = "Conversion failed";
  }
}

function swap() {
  const temp = fromCurrency.value;
  fromCurrency.value = toCurrency.value;
  toCurrency.value = temp;
  convert();
}

onMounted(() => {
  fetchCurrencies().then(convert);
});
watch([amount, fromCurrency, toCurrency], () => {
  convert();
});
</script>

<template>
  <div class="converter-ui">
    <div class="converter-row">
      <!-- Amount + From currency in one row -->
      <div class="input-combo">
        <input type="number" v-model.number="amount" class="combo-input" />
        <!-- From currency -->
        <div class="combo-select">
          <!-- On screen (flag + short code only) -->
          <img
            v-if="fromCurrency"
            :src="flagUrl(fromCurrency)"
            class="flag"
            alt="Flag"
          />
          <select v-model="fromCurrency" class="currency-select">
            <!-- Inside dropdown (code + name) -->
            <option
              v-for="currency in currencies"
              :key="currency.short_code"
              :value="currency.short_code"
            >
              {{ currency.short_code }}
            </option>
          </select>
        </div>
      </div>

      <!-- Swap -->
      <div class="icon swap-horizontal-vertical" @click="swap">
        <span class="arrow">&#8594;</span>
        <!-- right arrow on top -->
        <span class="arrow">&#8592;</span>
        <!-- left arrow below -->
      </div>

      <!-- Result + To currency in one row -->
      <div class="input-combo">
        <input class="combo-input" :value="result ?? ''" readonly />
        <!-- To currency -->
        <div class="combo-select">
          <img
            v-if="toCurrency"
            :src="flagUrl(toCurrency)"
            class="flag"
            alt="Flag"
          />
          <select v-model="toCurrency" class="currency-select">
            <option
              v-for="currency in currencies"
              :key="currency.short_code"
              :value="currency.short_code"
            >
              {{ currency.short_code }}
            </option>
          </select>
        </div>
      </div>
    </div>

    <div v-if="error" class="error">{{ error }}</div>
  </div>
</template>
