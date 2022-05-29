<template>
   <section class="menu">
      <div> {{ getCardQuantity() }} </div>

      <h2 class="menu__empty" v-if="discardPile.length === 0">Empty Discard-pile</h2>

      <div class="menu__grid" v-else>
         <div class="menu__cards" v-for="card, key in CardQuantity">

            <figure class="menu__card" v-if="key === 'KING' || key === 'QUEEN' || key === 'JACK' || key === 'ACE'">
               <img class="menu__card-image" :src="`https://deckofcardsapi.com/static/img/${key.charAt(0)}S.png`" :alt="card.code">
               <p class="menu__counter"> {{ card }} / 4 </p>
            </figure>

            <figure class="menu__card" v-else-if="key === '10'">
               <img class="menu__card-image" :src="`https://deckofcardsapi.com/static/img/${key.charAt(1)}S.png`" :alt="card.code">
               <p class="menu__counter"> {{ card }} / 4 </p>
            </figure>
            
            <figure class="menu__card" v-else>
               <img class="menu__card-image" :src="`https://deckofcardsapi.com/static/img/${key}S.png`" :alt="card.code">
               <p class="menu__counter"> {{ card }} / 4 </p>
            </figure>
         </div>
      </div>
      
      <button class="menu__discard-button" @click="toggleMenu">
         <img class="menu__discard-icon" src="/vector/discard_pile_icon.svg" alt="">

         <p class="menu__discard-length"> {{ discardPile.length }} </p>
      </button>
   </section>
</template>

<script>
export default {
   data() {
      return {
         CardQuantity: []
      }
   },

   props: {
      toggleMenu: Function,
      discardPile: Array
   },

   methods: {
      getCardQuantity() {
         let quantity = {};

         this.discardPile.forEach(card => {
            const value = card.value;
            quantity[value] = (quantity[value] || 0) + 1;
         })

         this.CardQuantity = quantity;
         console.log(this.CardQuantity)
      },
   }
}

</script>

<style>
   .menu {
      position: absolute;
      width: 100%;
      min-height: 100%;
      padding: 16rem 4rem;
      display: flex;
      justify-content: center;
      align-items: center;
      background: var(--highlight-color);
      z-index: 20;
   }

   .menu__empty {
      color: var(--primary-color);
   }

   .menu__grid {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
   }

   .menu__cards {
      display: flex;
      flex-direction: column;
      align-items: center;
   }

   .menu__card {
      display: flex;
      flex-direction: column;
      align-items: center;
      margin: var(--spacing-padding);
   }

   .menu__card-image {
      width: 100%;
      height: 30vh;
      border-radius: 5% 5% 5% 5%;
      box-shadow: 0 0.1rem 0.5rem var(--dark-color);
      margin-bottom: var(--spacing-padding);
   }

   .menu__counter {
      color: var(--primary-color);
      font-weight: 600;
      font-style: italic;
   }

   .menu__discard-button {
      widows: 4rem;
      height: 4rem;
      background: var(--highlight-color);
      position: absolute;
      bottom: 5%;
      border: solid black 1px;
      border-radius: 5px;
      padding: 0.5rem 1rem;
   }

   .menu__discard-button:hover {
      background: var(--secondary-color);
   }

   .menu__discard-icon {
      height: 90%;
   }

   .menu__discard-length {
      color: var(--primary-color);
      transform: translateX(-50%);
      top: 43%;
      left: 58%;
      position: absolute;
   }

   @media screen and (max-device-width: 767px) {

      .menu__card-image {
         height: 18vh;
      }

      .menu__discard-button {
         width: 12rem;
         height: 12rem;
         border-radius: 20px;
         font-size: 46px;
      }
   }
</style>