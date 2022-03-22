<template>
  <nav
    class="
      fixed
      inset-x-0
      z-10
      bg-white
      border-gray-200
      px-2
      sm:px-4
      py-2.5
      rounded
      dark:bg-gray-800
    "
  >
    <div class="container flex flex-wrap justify-between items-center mx-auto">
      <a href="#" class="flex items-center">
        <div class="logo w-10 h-10"></div>
        <span
          class="
            self-center
            text-xl
            font-semibold
            whitespace-nowrap
            dark:text-white
          "
          >ZoraUI</span
        >
      </a>
    </div>
  </nav>
</template>

<script lang="ts">
import Vue from 'vue'
export default Vue.extend({
  name: 'HeaderComponent',
})
</script>

<style >
.logo {
  box-sizing: border-box;
  margin: 0;
  min-width: 0;
  background: radial-gradient(
    75.29% 75.29% at 64.96% 24.36%,
    #dcc8d0 15.62%,
    #78c8cf 30.21%,
    #4d959e 42.71%,
    #305eb9 55.73%,
    #311f12 79.69%,
    #684232 90.62%,
    #2d1c13 100%
  );
  height: 30px;
  width: 30px;
  border-radius: 100%;
  margin-right: 10px;
}
</style>