<script>
import PokemonCardVue from "./PokemonCard.vue";

export default {
  components: {
    // Path: src/components/PokemonCard.vue
    PokemonCard: PokemonCardVue,
  },
  data() {
    return {
      showcase: {
        id: "swsh1-190",
        name: "Morpeko V",
        supertype: "Pokémon",
        subtypes: ["Basic", "V"],
        number: "190",
        rarity: "Rare Ultra",
        images: {
          small: "https://images.pokemontcg.io/swsh1/190.png",
          large: "https://images.pokemontcg.io/swsh1/190_hires.png",
        },
      },
    };
  },
};
</script>

<template>
  <PokemonCard
    :name="showcase.name"
    :img="showcase.images.large"
    :number="showcase.number"
    :supertype="showcase.supertype"
    :subtypes="showcase.subtypes"
    :rarity="showcase.rarity"
    :showcase="true"
  />
</template>

<style scoped></style>
