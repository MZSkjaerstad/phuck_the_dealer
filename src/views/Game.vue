<template>
   <main class="game">
      <section class="game__round" v-if="switches.gameFinished === false">
         <div class="game__drunk-o-meter">Computers drunk-o-meter  {{game.drunkOMeter}} / 10</div>

         <div class="game__finished" v-if="switches.roundFinished === true" >
            <div> {{ game.result }} </div>

            <img :src="game.currentCard.image" alt="">

            <button @click="fetchNewCard">New card</button>
         </div>

         <div class="game__round" v-if="switches.roundFinished === false">
            <div class="game__ongoing" v-if="switches.lastRound === false">
               <div>Guess the value of this card</div>

               <div>First round</div>
               
               <img class="game__card-back" src="/images/card_back.png" alt="">

               <input class="game__input" type="number" placeholder="input" v-model="game.input">

               <button @click="guessValue">Guess</button>
            </div>

            <div class="game__ongoing" v-if="switches.lastRound === true">
               <div> {{ game.result }} </div>

               <div>Last round</div>

               <img class="game__card-back" src="/images/card_back.png" alt="">

               <input class="game__input" type="number" placeholder="input" v-model="game.input">

               <button @click="guessValue">Guess</button>
            </div>
         </div>

         <div class="game__discard-button" @click="toggleMenu">
            <div> {{ discardPile.length }} </div>
         </div>
      </section>

      <section v-if="switches.gameFinished === true">
         FINISHED!
         <router-link :to="'/'">
            <button>Back to start</button>
         </router-link>
      </section>

     <DiscardPileMenu
         class="discard__menu" 
         v-if="switches.menuToggled === true" 
         :discardPile="discardPile" 
         :toggleMenu="toggleMenu" 
     />
   </main>
</template>

<script>
import DiscardPileMenu from '../components/DiscardPileMenu.vue'

export default {
   components: {
      DiscardPileMenu,
   },

   data() {
      return {
         deck: {
            id: null,
            remaining: null,
         },
         game: {
            currentCard: undefined,
            cardValue: null,
            input: 7,
            result: undefined,
            drunkOMeter: 0,
         },
         switches: {
            lastRound: true,
            roundFinished: true,
            gameFinished: false,
            menuToggled: false,
         },
         discardPile:[],
      }
   },

   async created() {
      this.fetchDeck()
         .then(this.fetchNewCard)
         .catch(err => console.log(error))
   },
   
   methods: {
      async fetchDeck() {
         const url = 'https://deckofcardsapi.com/api/deck/new/shuffle/?deck_count=1';
         const res = await fetch(url);
         const result = await res.json();

         this.deck.id = result.deck_id;
         this.deck.remaining = result.remaining;
         console.log ('[SUCCESS] New deck fetched =', result);
      },

      async fetchNewCard() {
         /* Push & clear previous card */
         if (this.game.currentCard !== undefined) {
            this.discardPile.push(this.game.currentCard);
            console.log('[SUCCESS] Moved old card into discardPile =', this.discardPile)
         }

         /* Fetch new card */
         const url = `https://deckofcardsapi.com/api/deck/${this.deck.id}/draw/?count=1`;
         const res = await fetch(url);
         const result = await res.json();

         this.game.currentCard = result.cards[0];
         this.game.cardValue = this.filterValue()
         this.deck.remaining = result.remaining;

         /* Toggle switches */
         this.switches.lastRound = !this.switches.lastRound
         this.switches.roundFinished = !this.switches.roundFinished;

         console.log('[SUCCESS] Updated currentCard with new card =', this.game.currentCard)
         console.log('[NOTE]', this.deck.remaining, 'cards remaining...')
      },

      filterValue() {
         let initialValue = this.game.currentCard.value;

         if(initialValue === "KING") {
            initialValue = 13;
         }
         else if(initialValue === "QUEEN") {
            initialValue = 12;
         }
         else if(initialValue === "JACK") {
            initialValue = 11;
         }
         else if(initialValue === "ACE") {
            initialValue = 1;
         } else {
            initialValue = parseInt(initialValue)
         }
         return initialValue;
      },

      guessValue() {
         /* [ FIRST ROUND ] */
         if (this.switches.lastRound === false) {
            
            console.log('[FIRST ROUND]')
            /* If input is higher */
            if (this.game.input > this.game.cardValue) {
               this.game.result = 'Lower!';
               this.switches.lastRound = !this.switches.lastRound
            }

            /* If input is Lower */
            if (this.game.input < this.game.cardValue) {
               this.game.result = 'Higher!';
               this.switches.lastRound = !this.switches.lastRound
            }

            /* If input is Correct */
            if (this.game.input === this.game.cardValue) {
               this.game.drunkOMeter++;
               this.game.result = this.drunkOMeterResponse()
               this.switches.lastRound = !this.switches.lastRound
               this.switches.roundFinished = !this.switches.roundFinished;
            }
         }

         /* [ LAST ROUND ] */
         else {
            console.log('[LAST ROUND]')
            /* If input is higher */
            if (this.game.input > this.game.cardValue) {
               this.game.result =  `Wrong! Drink ${this.game.input - this.game.cardValue}.`;
            }

            /* If input is Lower */
            if (this.game.input < this.game.cardValue) {
               this.game.result = `Wrong! Drink ${this.game.cardValue - this.game.input}.`;
            }

            /* If input is Correct */
            if (this.game.input === this.game.cardValue) {
               this.game.drunkOMeter++;
               this.game.result = this.drunkOMeterResponse()
            }

         this.switches.roundFinished = !this.switches.roundFinished;
         }

         this.checkIfFinished()
      },

      drunkOMeterResponse() {
         let drunkOMeterResponse = undefined;

         if (this.game.drunkOMeter === 1) {
            drunkOMeterResponse = 'correct! Computer takes a sip';

         } else if (this.game.drunkOMeter === 2) {
            drunkOMeterResponse = 'correct again! Computer takes another sip';

         } else if (this.game.drunkOMeter === 3) {
            drunkOMeterResponse = 'correct! This drink is delish!';

         } else if (this.game.drunkOMeter === 4) {
            drunkOMeterResponse = 'correct once again! Computer is feeling the tingles hehe';

         } else if (this.game.drunkOMeter === 5) {
            drunkOMeterResponse = 'correct.. Damn it! One more sip for computer';

         } else if (this.game.drunkOMeter === 6) {
            drunkOMeterResponse = '*hic*.. c-correct.';

         } else if (this.game.drunkOMeter === 7) {
            drunkOMeterResponse = '.. you kn..*hic*..ow computer looves you man!';

         } else if (this.game.drunkOMeter === 8) {
            drunkOMeterResponse = 'Heyy hoo, down the hatch it goooess trallalla!';

         } else if (this.game.drunkOMeter === 9) {
            drunkOMeterResponse = '1110001001 110011001010 0101..';

         } else {
            drunkOMeterResponse = 'zzzzzzzzzzzzzz...';
         }

         return drunkOMeterResponse;
      },

      checkIfFinished() {
         if (this.game.drunkOMeter === 10) {
            this.switches.gameFinished = !this.switches.gameFinished;
         }

         if (this.discardPile.length === 52) {
            this.switches.gameFinished = !this.switches.gameFinished;
         }
      },

      toggleMenu() {
         this.switches.menuToggled = !this.switches.menuToggled;
      },

      consoleLog() {
         console.log('Input', this.game.input)
         console.log('Card Value', this.game.cardValue)
         console.log('The result is', this.game.result)
         console.log('DrunkOMeter', this.game.drunkOMeter)
         //console.log('Last round', this.game.lastRound)
         //console.log('Round finished', this.game.roundFinished)
      },
   }, 
}
</script>

<style>
   .game {
      width: 100%;
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
      z-index: 10;
   }

   .game__drunk-o-meter {
      position: absolute;
      top: 5%
   }

   .game__round {
      display: flex;
      flex-direction: column;
      align-items: center;
   }

   .game__card-back {
      height: 20rem;
   }

   .game__ongoing {
      display: flex;
      flex-direction: column;
      align-items: center;
   }

   .game__finished {
      display: flex;
      flex-direction: column;
      align-items: center;
   }

   .game__input {
      text-align: center;
   }

   .game__discard-button {
      position: absolute;
      background: lightgray;
      bottom: 5%;
      border: solid black 1px;
      border-radius: 5px;
      padding: 0.6rem 1rem;
   }
</style>