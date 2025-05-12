<script setup>
import OrderSystem from './components/OrderSystem.vue'
</script>

<template>
  <div class="app">
    <header>
      <h1>McDonald's Order Management System</h1>
    </header>
    <OrderSystem />
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: Arial, sans-serif;
  line-height: 1.6;
  background-color: #f0f2f5;
}

.app {
  min-height: 100vh;
}

header {
  background-color: #fff;
  padding: 20px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  margin-bottom: 20px;
}

h1 {
  color: #333;
  text-align: center;
}
</style>
