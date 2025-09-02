<template>
  <div id="app" class="container">
    <CurrencyConverter />
    <CurrencyList />
  </div>
</template>

<script setup>
import CurrencyConverter from "./components/CurrencyConverter.vue";
import CurrencyList from "./components/CurrencyList.vue";
</script>

<style>
.container {
  max-width: 900px;
  margin: 2rem auto;
  font-family: Arial, sans-serif;
  padding: 0 1rem;
}
</style>
