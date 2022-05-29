<template>
   <section class="home">
      <img class="game__logo" src="/vector/game_logo.svg" alt="">
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
      flex-direction: column;
      justify-content: center;
      align-items: center;
   }

   .game__logo {
      width: auto;
      height: 50%;
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