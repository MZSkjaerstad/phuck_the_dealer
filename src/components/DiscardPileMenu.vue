<template>
   <section class="menu">
      <div> {{ getCardQuantity() }} </div>

      <div class="menu__grid">

         <div class="menu__cards" v-for="card, key in CardQuantity">

            <figure>
               <img :src="`https://deckofcardsapi.com/static/img/${key.charAt(0)}S.png`" :alt="card.code">
            </figure>
            
            <div> {{ card }} / 4 </div>
         </div>
      </div>
      
      <div class="menu__discard-button" @click="toggleMenu">
         {{ discardPile.length }}
      </div>
   </section>
</template>

<script>
export default {
   data() {
      return {
         CardQuantity: {}
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

      sortCardQuantity() {
         
      }
   }
}

</script>

<style>
   .menu {
      position: absolute;
      width: 100%;
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
      background: lightgray;
      z-index: 20;
   }

   .menu__grid {
      display: flex;
      flex-wrap: wrap;
   }

   .menu__cards {
      display: flex;
      flex-direction: column;
      align-items: center;

   }

   .menu__discard-button {
      position: absolute;
      bottom: 5%;
      border: solid black 1px;
      border-radius: 5px;
      padding: 0.6rem 1rem;
   }
</style>