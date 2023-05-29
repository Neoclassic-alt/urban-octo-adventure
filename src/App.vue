<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
import Logo from './components/Logo.vue'
</script>

<template>
  <header class="header">
    <Logo />
    <div class="header__links">
      <RouterLink to="/" class="header__link">Форма</RouterLink>
      <RouterLink to="/info" class="header__link">Превью</RouterLink>
    </div>
    <div></div>
  </header>

  <RouterView />
</template>

<style scoped>
.header {
  display: flex;
  justify-content: space-between;
  padding-top: 22px;
  padding-bottom: 22px;
  box-shadow: 0px 1px 0px rgba(17, 17, 17, 0.1);
}
.header__links {
  display: flex;
  width: 133px;
  justify-content: space-between;
}
.header__link {
  text-decoration: none;
  color: rgba(17, 17, 17, 0.48);
}
</style>
