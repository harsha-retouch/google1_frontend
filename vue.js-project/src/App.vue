<template>
  <header>
  

    <div class="wrapper">
    </div>
  </header>

  <main>
    <Froms />
  </main>
</template>

<script setup>
import Froms from './form.vue'
</script>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

header h1 {
  text-align: center;
  color: #0d7fb3;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>