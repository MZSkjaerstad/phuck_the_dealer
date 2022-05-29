<template>
   <section class="home">
      <router-link :to="'/game'">
         <button class="home__button">New Game</button>
      </router-link>
   </section>
</template>

<script>
// import { defineComponent } from 'vue'

export default {
   components: {

   }
}
</script>

<style>
   .home {
      width: 100vw;
      height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
   }

   .home__button {
      font-size: var(--font-size-input);
      font-weight: 600;
      font-style: italic;
      text-transform: uppercase;
      margin-top: var(--spacing-medium);
      padding: var(--spacing-padding) var(--spacing-small);
      box-shadow: 0 0.2rem 1rem var(--dark-color);
      border-radius: 5px;
   }

</style>