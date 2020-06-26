<template>
  <section>
    <div class="selected-ingredients-section row justify-content-center p-3">
      <div class="col-12 col-md-auto mr-0 mr-md-3">
        Selected ingredients
        <div class="row justify-content-center">
          <div class="ingredient-button d-flex flex-column justify-content-center align-items-center"
            v-for="(ingredient, index) of selectedIngredients" :key="index"
            :class="ingredient ? 'filled' : 'empty'" @click="removeIngredient(index)">
            <span v-if="ingredient" class="ingredient-icon">
              <img :src="getIcon(ingredient.name)" alt="">
            </span>          
            <div v-if="ingredient" class="ingredient-title">
              {{ingredient.name}}
            </div>
          </div>          
        </div>
        <small>Tap ingredient to remove</small>
      </div>
      <div class="col-12 col-md-auto mt-3 mt-md-0 ml-md-3c output-section">
        Expected output
        <div class="row justify-content-center">
          <div class="ingredient-button d-flex flex-column justify-content-center align-items-center"
            v-for="(result, index) of expectedOutput" :key="index">
            <span v-if="result" class="ingredient-icon">
              <img :src="getIcon(result.name)" alt="">
            </span>          
            <div v-if="result" class="ingredient-title">
              {{result.name}}
            </div>
            <div v-if="result" class="output-chance">
              {{result.chance}}%
            </div>
          </div>               
        </div>        
      </div>
    </div>

    <div class="row form-inline mb-3 justify-content-center">
      <div class="col-auto form-group">
        <label class="mr-md-2" for="sortby">Sort by:</label>
        <select class="form-control form-control-sm" id="sortby" v-model="order">
          <option :value="0">Name &darr;</option>
          <option :value="1">Name &uarr;</option>
          <option :value="2">Value &darr;</option>
          <option :value="3">Value &uarr;</option>
          <option :value="4">Type Affinity</option>
        </select>
      </div>
      <div class="col-auto form-group">
        <label class="mr-md-2" for="filterby">Filter by:</label>
        <select class="form-control form-control-sm" id="filterby" v-model="filter">
          <option :value="0">None</option>
          <option :value="1">Apricorns</option>
        </select>
      </div>
    </div>

    <div class="ingredient-section">
      <div class="row justify-content-center" >
        <div class="ingredient-button col-auto d-flex flex-column justify-content-center align-items-center"
          v-for="ingredient of filteredIngredients" :key="ingredient.name"
          :class="ingredient.type"
          @click="pickIngredient(ingredient)">
          <span class="ingredient-power">
            {{ingredient.crampower}}
          </span>
          <span class="ingredient-icon">
            <img :src="getIcon(ingredient.name)" alt="">
          </span>          
          <div class="ingredient-title">
            {{ingredient.name}}
          </div>
          <span class="ingredient-marks" v-if="isIngredientPicked(ingredient)">
            <span :class="selectedIngredient1 === ingredient ? 'selected' : ''">
              &bull;
            </span>
            <span :class="selectedIngredient2 === ingredient ? 'selected' : ''">
              &bull;
            </span>
            <span :class="selectedIngredient3 === ingredient ? 'selected' : ''">
              &bull;
            </span>
            <span :class="selectedIngredient4 === ingredient ? 'selected' : ''">
              &bull;
            </span>                                    
          </span>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import ingredients from "../data/ingredients";
import recipes from "../data/recipes";
import fixedRecipes from "../data/fixed-recipes";
import apricornRecipes from "../data/apricorn-recipes";
import icons from "../data/icons";

const recipeRangeMin = [0,1,21,31,41,51,61,71,81,91,101,111,121,131,141,151];
const recipeRangeMax = [0,20,30,40,50,60,70,80,90,100,110,120,130,140,150,999];
const apricornTierChance = [0, 24.7, 24.7, 24.7, 24.7, 1, 0.1, 0.1];

const OrderBy = {
  NAME_ASC: 0,
  NAME_DESC: 1,
  VALUE_ASC: 2,
  VALUE_DESC: 3,
  TYPE: 4
}

export default {
  name: 'Main',
  created() {
    console.log("Started");
  },
  data() {
    return {
      ingredients: ingredients,
      selectedIngredient1: null,
      selectedIngredient2: null,
      selectedIngredient3: null,
      selectedIngredient4: null,
      expectedOutput: [null],
      filter: 0,
      order: 0
    }
  },
  methods: {
    clearOutput() {
      this.expectedOutput = [null];
    },
    isIngredientPicked(ingredient) {
      return this.selectedIngredients.includes(ingredient);
    },
    allIngredientsPicked() {
      return (
        this.selectedIngredient1 != null &&
        this.selectedIngredient2 != null &&
        this.selectedIngredient3 != null &&
        this.selectedIngredient4 != null
      );
    },
    pickIngredient(ingredient) {
      let changed = false;

      if(this.selectedIngredient1 == null) {
        this.selectedIngredient1 = ingredient;
        changed = true;
      } else if(this.selectedIngredient2 == null) {
        this.selectedIngredient2 = ingredient;
        changed = true;
      } else if(this.selectedIngredient3 == null) {
        this.selectedIngredient3 = ingredient;
        changed = true;
      } else if(this.selectedIngredient4 == null) {
        this.selectedIngredient4 = ingredient;
        changed = true;
      }

      if(changed) {
        this.onIngredientPicked(ingredient);
      }
    },
    removeIngredient(index) {
      switch(index) {
        case 0:
          this.selectedIngredient1 = null;
          break;
        case 1:
          this.selectedIngredient2 = null;
          break;
        case 2:
          this.selectedIngredient3 = null;
          break;
        case 3:
          this.selectedIngredient4 = null;
          break;                              
      }
      this.clearOutput();
    },
    onIngredientPicked(ingredient) {
      if(this.allIngredientsPicked()) {
        console.log("All Ingredients Picked!")

        const isApricornOutput = this.selectedIngredients
          .filter(x => x.name.includes("Apricorn")).length === 4;

        // CASE: ALL APRICORNS
        if(isApricornOutput) {
          console.log("Using apricorn method");

          const finalTable = {
            tier1: [],
            tier2: [],
            tier3: [],
            tier4: [],
            tier5: [],
            tier6: [],
            tier7: [],
          }

          this.selectedIngredients.map(ingredient => {
            return apricornRecipes.find(recipe => recipe.item === ingredient.name);
          }).forEach(table => {
            for(let i=1; i<=7; i++) {
              const tierChance = apricornTierChance[i];
              finalTable[`tier${i}`].push({
                item: table[`tier${i}`],
                chance: tierChance / 4.0
              });
            }
          });

          const output = [];

          for(const key in finalTable) {
            const mergedTier = finalTable[key].reduce((accumulator, current) => {
              const duplicate = accumulator.find(x => x.item === current.item);

              if(duplicate) {
                duplicate.chance += current.chance;
              } else {
                accumulator.push(current);
              }

              return accumulator;

            }, []);

            mergedTier.forEach(result => output.push({
              name: result.item,
              chance: result.chance.toFixed(1)
            }))
          }

          this.expectedOutput = output;

        } else {
          const fixedRecipe = fixedRecipes.find(recipe => {
            return this.selectedIngredient1.name === recipe.item1 &&
            this.selectedIngredient3.name === recipe.item3 &&
            this.selectedIngredient4.name === recipe.item4;
          })

          // CASE: FIXED RECIPE
          if(fixedRecipe != null) {
            console.log("Using fixed recipe method");
            this.expectedOutput = [{
              name: fixedRecipe.output,
              chance: 100
            }]
          } else {
            // CASE: RECIPE POOL
            console.log("Using recipe pool method")
            const recipe = recipes.find(recipe => recipe.pool === this.selectedIngredient1.type);
            const totalCrampower = this.selectedIngredients
              .map(x => x.crampower)
              .reduce((a,b) => a+b);
    
            for(let range=1; range<=15; range++) {
              if(totalCrampower >= recipeRangeMin[range] && totalCrampower <= recipeRangeMax[range]) {
                const itemName = recipe[`range${range}`];

                if(itemName === "<sweet_rewards1>") {
                  this.expectedOutput = [
                    {name: "Strawberry Sweet", chance: "???"},
                    {name: "Love Sweet", chance: "???"},
                    {name: "Berry Sweet", chance: "???"},
                    {name: "Clover Sweet", chance: "???"},
                    {name: "Flower Sweet", chance: "???"},
                  ]
                } else if(itemName === "<sweet_rewards2>") {
                  this.expectedOutput = [
                    {name: "Strawberry Sweet", chance: "???"},
                    {name: "Ribbon Sweet", chance: "???"},
                    {name: "Star Sweet", chance: "???"}
                  ]
                } else {
                  this.expectedOutput = [ {name: itemName, chance: 100 }];
                }
                break;
              }
            }
          }
        }

      } else {
        this.clearOutput();
      }
    },
    getPoolClass(pool) {
      return pool === this.recipePool ? 'pool-selected' : '';
    },
    getRangeClass(pool, range) {
      const minRange = recipeRangeMin[range];
      const maxRange = recipeRangeMax[range];
      return this.totalCrampower >= minRange && this.totalCrampower <= maxRange ? 'range-selected' : '';
    },
    getIcon(itemName) {
      const iconData = icons.find(x => x.name === itemName);
      if(iconData) {
        return `icon/${iconData.icon}`;
      } else {
        return "icon/unknown-item.png";
      }
    }
  },
  computed: {
    selectedIngredients() {
      return [
        this.selectedIngredient1,
        this.selectedIngredient2,
        this.selectedIngredient3,
        this.selectedIngredient4,
      ];
    },
    filteredIngredients() {
      let filteredList = this.ingredients.filter(item => {
        switch(this.filter) {
          case 1:
            return item.name.includes("Apricorn");
          default:
          case 0:
            return true;
        }
      });

      switch(this.order) {
        default:
        case OrderBy.NAME_ASC:
          filteredList.sort((a,b) => a.name.localeCompare(b.name));
          break;      
        case OrderBy.NAME_DESC:
          filteredList.sort((a,b) => b.name.localeCompare(a.name));
          break;                
        case OrderBy.VALUE_ASC:
          filteredList.sort((a,b) => b.crampower - a.crampower);
          break;          
        case OrderBy.VALUE_DESC:
          filteredList.sort((a,b) => a.crampower - b.crampower);
          break;
        case OrderBy.TYPE:
          filteredList.sort((a,b) => a.type.localeCompare(b.type));
          break;
      }

      return filteredList;
    }
  }
}
</script>


<style scoped>
.ingredient-button {
  position: relative;
  border: 1px solid black;
  height: 65px;
  width: 65px;
  cursor: pointer;
  margin-right: 5px;
  margin-bottom: 5px;
  border-radius: 3px;
  overflow: hidden;
  line-height: normal;
  user-select: none;
  font-weight: bold;
}
.output-section .ingredient-button {
  cursor: auto;
}

.ingredient-title {
  font-size: 10px;
}
.ingredient-power {
  display: block;
  position: absolute;
  top: 0;
  right: 2px;
  font-size: 11px;
  font-weight: bold;
}
.output-chance {
  display: block;
  position: absolute;
  top: 0;
  right: 2px;
  font-size: 10px;
  font-weight: normal;
}
.ingredient-type {
  display: block;
  position: absolute;
  bottom: 0;
  left: 2px;
  font-size: 10px;
}
.ingredient-icon {
  min-height: 32px;
  min-width: 32px;
}

.ingredient-section .ingredient-title {
  color: white;
  text-shadow:
   -1px -1px 0 rgba(0,0,0,1),  
    1px -1px 0 rgba(0,0,0,1),
    -1px 1px 0 rgba(0,0,0,1),
     1px 1px 0 rgba(0,0,0,1);
}
.ingredient-marks {
  display: block;
  position: absolute;
  top: 0;
  left: 2px;
  font-size: 10px;
  color: #777;
}
.ingredient-marks .selected{
  color: white;
}

.selected-ingredients-section .filled {
  background-color: white;
}
.selected-ingredients-section .empty {
  background-color: #ddd;
}

.selected-ingredients-section small {
  font-size: 12px;
  font-style: italic;
}

.filter-section {
  font-size: 12px;
  text-align: left;
  user-select: none;
}

.filter-title {
  font-weight: bold;
}

.filter-option {
  cursor: pointer;
}

.Bug { background-color: #C6D16E; }
.Dark { background-color: #A29288; }
.Dragon { background-color: #A27DFA; }
.Electric { background-color: #FAE078; }
.Fairy { background-color: #F4BDC9; }
.Fighting { background-color: #D67873; }
.Fire { background-color: #F5AC78; }
.Flying { background-color: #C6B7F5; }
.Ghost { background-color: #A292BC; }
.Grass { background-color: #A7DB8D; }
.Ground { background-color: #EBD69D; }
.Ice { background-color: #BCE6E6; }
.Normal { background-color: #C6C6A7; }
.Poison { background-color: #C183C1; }
.Psychic { background-color: #FA92B2; }
.Rock { background-color: #D1D1E0; }
.Steel { background-color: #9DB7F5; }
.Water { background-color: #9DB7F5; }
</style>
