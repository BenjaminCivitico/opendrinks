<template>
  <div id="recipe-find">
    <b-form-input @keydown.enter="onEnter" @input="onInput"></b-form-input>

    <b-form-group class="mt-2" label="Filter by:">
      <b-form-radio-group
        v-model="searchParameters"
        name="searchParameters"
      >
        <b-form-radio value="name">Name</b-form-radio>
        <b-form-radio
          v-b-tooltip.hover
          title="You can search for multiple ingredients by seperating them with a space"
          value="ingredients"
        >Ingredients</b-form-radio>
        <b-form-radio
          v-b-tooltip.hover
          title="You can search for multiple keywords by seperating them with a space"
          value="keywords"
        >Keywords</b-form-radio>
      </b-form-radio-group>
    </b-form-group>

    <b-list-group class="mt-4" v-if="search.length > 0">
      <b-list-group-item
        v-for="o in filterResults"
        v-bind:key="o.name"
        :href="'/recipe/' + o.filename"
      >{{ o.name }}</b-list-group-item>
    </b-list-group>
  </div>
</template>

<script>
import recipes from '../recipes';

const LIMIT = 50; // Limit filtered results.

export default {
  name: 'RecipeFind',
  data() {
    return {
      data: [],
      searchParameters: ['name'],
      search: '',
    };
  },
  computed: {
    filterResults() {
      const searchParts = this.search.toLowerCase().split(' ');
      const isNameEnabled = this.searchParameters.includes('name');
      const isIngredientsEnabled = this.searchParameters.includes('ingredients');
      const isKeywordsEnabled = this.searchParameters.includes('keywords');

      let filtered = [];

      if (isNameEnabled) {
        filtered = this.data
          .filter(recipe => recipe.name.toLowerCase().indexOf(this.search.toLowerCase()) > -1);
      }

      if (isIngredientsEnabled) {
        filtered = this.data
          .filter(recipe => recipe.ingredients
            .some(i => searchParts.includes(i.ingredient.toLowerCase())));
      }

      if (isKeywordsEnabled) {
        filtered = this.data
          .filter(recipe => recipe.keywords && recipe.keywords
            .some(i => searchParts.includes(i.toLowerCase())));
      }

      // Sort the results.
      filtered = filtered.sort((recipeA, recipeB) => {
        // Name-matches to the top if the user wanted to find a drink based on the name
        const hasName = recipeB.name.toLowerCase().indexOf(this.search.toLowerCase()) > -1 ? 1 : -1;
        return !isNameEnabled ? 0 : hasName;
      });

      return filtered.slice(0, LIMIT);
    },
  },
  created() {
    const data = recipes.getRecipes();
    this.data = data;
    window.document.title = 'Open Drinks - Search';
  },
  methods: {
    onEnter() {
      this.$router.push({
        name: 'recipe',
        params: { id: this.filterResults[0].filename },
      });
    },
    onInput(evt) {
      this.search = evt;
    },
  },
};
</script>
