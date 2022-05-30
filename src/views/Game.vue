<template>
   <main class="game">
      <!-- Game ongoing -->
      <section class="game__round" v-if="switches.gameFinished === false">
         <div class="game__drunk-o-meter">Computers drunk-o-meter  {{game.drunkOMeter}} / 10</div>
         
         <!-- Round finished -->
         <div class="game__finished" v-if="switches.roundFinished === true" >
            <h2 class="game__message"> {{ game.result }} </h2>

            <img class="game__card" :src="game.currentCard.image" alt="">
            
            <p class="game_phase-tracker">round finished</p>

            <button class="game__new-card" @click="fetchNewCard">New card</button>
         </div>

         <!-- Round ongoing -->
         <div class="game__round" v-if="switches.roundFinished === false">

            <!-- First round -->
            <div class="game__ongoing" v-if="switches.lastRound === false">
               <h2 class="game__message">Guess the value of this card</h2>
               
               <img class="game__card" src="/images/card_back.png" alt="">

               <p class="game_phase-tracker">number 7 is smart</p>

               <div class="game__controller">
                  <div class="game__input-container">
                     <button class="game__remove-value" @click="removeValue">-</button>

                     <input class="game__input" type="number" placeholder="input" min="1" max="13" v-model="game.input">

                     <button class="game__add-value" @click="addValue">+</button>
                  </div>

                  <button class="game__guess" @click="guessValue">Guess</button>
               </div>
            </div>

            <!-- Last round -->
            <div class="game__ongoing" v-if="switches.lastRound === true">
               <h2 class="game__message"> {{ game.result }} </h2>

               <img class="game__card" src="/images/card_back.png" alt="">

               <div class="game_phase-tracker">Guess again</div>

               <div class="game__controller">
                  <div class="game__input-container">
                     <button class="game__remove-value" @click="game.input -= 1">-</button>

                     <input class="game__input" type="number" placeholder="input" min="1" max="13" v-model="game.input">

                     <button class="game__add-value" @click="game.input += 1">+</button>
                  </div>

                  <button class="game__guess" @click="guessValue">Guess</button>
               </div>
            </div>
         </div>

         <!-- Discardpile -->
         <button class="game__discard-button" @click="toggleMenu">
            <img class="game__discard-icon" src="/vector/discard_pile_icon.svg" alt="">

            <p class="game__discard-length"> {{ discardPile.length }} </p>
         </button>
      </section>

      <!-- Game finished -->
      <section class="game__finished" v-if="switches.gameFinished === true">
         <h2>FINISHED!</h2>
         <router-link :to="'/'">
            <button class="game__to-start">Back to start</button>
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

   mounted() {
      console.log(this.$route)
   },
   
   methods: {
      async fetchDeck() {
         const url = 'https://deckofcardsapi.com/api/deck/new/shuffle/?deck_count=1';
         const response = await fetch(url);
         try {
            await this.handleDeckFetch(response);
         } catch(error) {
            this.error = error.message;
         }
      },

      async handleDeckFetch(response) {
         if (response.status >= 200 && response.status < 300) {
            const result = await response.json();
            this.deck.id = result.deck_id;
            this.deck.remaining = result.remaining;
            console.log ('[SUCCESS] New deck fetched =', result);
         } else {
            if(response.status === 404) {
               throw new Error('Url ikke funnet.');
               }
               if(response.status === 401) {
                  throw new Error('Ikke authorisert.');
               }
               if(response.status > 500) {
                  throw new Error('Server error.');
               }
            throw new Error('Her gikk noe galt.');
         }
      },

      async fetchNewCard() {
         /* Push & clear previous card */
         if (this.game.currentCard != undefined) {
            this.discardPile.push(this.game.currentCard);
            console.log('[SUCCESS] Moved old card into discardPile =', this.discardPile)

            this.checkIfFinished()
         }

         /* Fetch new card */
         const url = `https://deckofcardsapi.com/api/deck/${this.deck.id}/draw/?count=1`;
         const response = await fetch(url);
         try {
            await this.handleCardFetch(response)
         } catch(error) {
            this.error = error.message;
         }
      },

      async handleCardFetch(response) {
         if (response.status >= 200 && response.status < 300) {
            const result = await response.json();
            this.game.currentCard = result.cards[0];
            this.game.cardValue = this.filterValue()
            this.deck.remaining = result.remaining;

            /* Toggle switches */
            this.switches.lastRound = !this.switches.lastRound
            this.switches.roundFinished = !this.switches.roundFinished;

            console.log('[SUCCESS] Updated currentCard with new card =', this.game.currentCard)
            console.log('[NOTE]', this.deck.remaining, 'cards remaining...')
         } else {
            if(response.status === 404) {
               throw new Error('Url ikke funnet.');
               }
               if(response.status === 401) {
                  throw new Error('Ikke authorisert.');
               }
               if(response.status > 500) {
                  throw new Error('Server error.');
               }
            throw new Error('Her gikk noe galt.');
         }
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

      addValue() {
         if (this.game.input != 13) {
            this.game.input += 1;
         }
      },

      removeValue() {
         if (this.game.input != 1) {
            this.game.input -= 1;
         }
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
               this.game.result =  `Wrong! Drink ${this.game.input - this.game.cardValue}`;
            }

            /* If input is Lower */
            if (this.game.input < this.game.cardValue) {
               this.game.result = `Wrong! Drink ${this.game.cardValue - this.game.input}`;
            }

            /* If input is Correct */
            if (this.game.input === this.game.cardValue) {
               this.game.drunkOMeter++;
               this.game.result = this.drunkOMeterResponse()
            }

         this.switches.roundFinished = !this.switches.roundFinished;
         }
      },

      drunkOMeterResponse() {
         let drunkOMeterResponse = undefined;

         if (this.game.drunkOMeter === 1) {
            drunkOMeterResponse = 'Correct! Computer takes a sip';

         } else if (this.game.drunkOMeter === 2) {
            drunkOMeterResponse = 'Correct again! Computer takes another sip';

         } else if (this.game.drunkOMeter === 3) {
            drunkOMeterResponse = 'Correct! Computer glugs a huge one!';

         } else if (this.game.drunkOMeter === 4) {
            drunkOMeterResponse = 'Correct once again! Computer is beginning to feel the tingles..';

         } else if (this.game.drunkOMeter === 5) {
            drunkOMeterResponse = 'Correct.. Damn it! One more sip for computer';

         } else if (this.game.drunkOMeter === 6) {
            drunkOMeterResponse = '*Hic*.. C-correct.';

         } else if (this.game.drunkOMeter === 7) {
            drunkOMeterResponse = 'Again?! ..you tart piece of *hic*-ory!';

         } else if (this.game.drunkOMeter === 8) {
            drunkOMeterResponse = 'Heyy hoo, down the.. *hic* ..hatch it goooess..';

         } else if (this.game.drunkOMeter === 9) {
            drunkOMeterResponse = 'zzzZZzzZZzZzZZzzz..';

         } else {
            drunkOMeterResponse = '1110001001 110011001010 0101..';
         }

         return drunkOMeterResponse;
      },

      checkIfFinished() {
         if (this.game.drunkOMeter === 10 || this.discardPile.length === 52) {
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
      padding: var(--spacing-medium);
   }

   /* Drunk-o-meter */
   .game__drunk-o-meter {
      position: absolute;
      top: 5%
   }

   /* Game */
   .game__round {
      display: flex;
      flex-direction: column;
      align-items: center;
   }

   .game__message {
      text-align: center;
   }

   .game__card {
      height: 18rem;
      margin: var(--spacing-medium) 0;
      border-radius: 5% 5% 5% 5%;
      box-shadow: 0 0.2rem 1rem var(--dark-color);
   }

   .game_phase-tracker {
      font-weight: 600;
      font-style: italic;
      text-transform: uppercase;
      padding-bottom: var(--spacing-small);
   }

   /* Phases */
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

   /* Controller */
   .game__controller {
      box-shadow: 0 0.2rem 1rem var(--dark-color);
      border-radius: 30% 30% 30% 30%;
   }

   .game__input {
      width: 3rem;
      text-align: center;
      color: var(--primary-color);
      font-size: var(--font-size-input);
      background: var(--highlight-color);
      padding: var(--spacing-padding);
      border: none;
   }

   .game__remove-value {
      font-size: var(--font-size-input);
      font-weight: 600;
      padding: var(--spacing-padding) var(--spacing-small);
      border-right: var(--border-style) var(--primary-color);
      border-radius: 30% 0 0 0;
   }

   .game__add-value {
      font-size: var(--font-size-input);
      font-weight: 600;
      padding: var(--spacing-padding) var(--spacing-small);
      border-left: var(--border-style) var(--primary-color);
      border-radius: 0 30% 0 0;
   }

   .game__guess {
      width: 100%;
      font-size: var(--font-size-input);
      font-weight: 600;
      font-style: italic;
      text-transform: uppercase;
      padding: var(--spacing-padding) var(--spacing-small);
      border-top: var(--border-style) var(--primary-color);
      border-radius: 0 0 30% 30%;
   }

   .game__new-card {
      font-size: var(--font-size-input);
      font-weight: 600;
      font-style: italic;
      text-transform: uppercase;
      padding: var(--spacing-padding) var(--spacing-small);
      border-radius: 15% 15% 15% 15%;
      box-shadow: 0 0.2rem 1rem var(--dark-color);
   }

   /* Discard */
   .game__discard-button {
      widows: 4rem;
      height: 4rem;
      position: absolute;
      bottom: 5%;
      border: solid black 1px;
      border-radius: 5px;
      padding: 0.5rem 1rem;
   }

   .game__discard-icon {
      height: 90%;
   }

   .game__discard-length {
      color: var(--primary-color);
      transform: translateX(-50%);
      top: 43%;
      left: 58%;
      position: absolute;
   }

   /* Finished */

   .game__finished {
      display: flex;
      flex-direction: column;
      justify-content: center;
   }

   .game__to-start {
      font-size: var(--font-size-input);
      font-weight: 600;
      font-style: italic;
      text-transform: uppercase;
      margin-top: var(--spacing-medium);
      padding: var(--spacing-padding) var(--spacing-small);
      box-shadow: 0 0.2rem 1rem var(--dark-color);
      border-radius: 5px;
   }

   @media screen and (max-device-width: 767px) {
      .game__card {
         height: 46rem;
      }

      .game__input {
         width: 9rem;
      }

      .game__discard-button {
         width: 12rem;
         height: 12rem;
         right: 5%;
         border-radius: 20px;
         font-size: 46px;
      }
   }
</style>