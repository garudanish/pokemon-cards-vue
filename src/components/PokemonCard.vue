<template>
  <div
    class="card"
    :class="{
      active: this.active,
      interacting: this.interacting,
      loading: this.loading,
    }"
    :data-number="number"
    :data-subtypes="subtypes"
    :data-supertype="supertype"
    :data-rarity="rarity"
    :data-gallery="gallery"
    :style="styles"
    bind:this="{thisCard}"
  >
    <div class="card__translater">
      <button
        class="card__rotator"
        bind:this="{rotator}"
        @click="activate"
        @pointermove="interact"
        @mouseout="interactEnd"
        @blur="deactivate"
        :aria-label="`Expand the Pokemon Card; ${name}.`"
        tabindex="0"
      >
        <img
          class="card__back"
          :src="back_img"
          alt="The back of a Pokemon Card, a Pokeball in the center with Pokemon logo above and below"
          loading="lazy"
          width="660"
          height="921"
        />
        <div class="card__front">
          <img
            :src="front_img"
            :alt="`Front design of the ${name} Pokemon Card, with the stats and info around the edge`"
            v-on:load="imageLoader"
            loading="lazy"
            width="660"
            height="921"
          />
          <CardShine :subtypes="subtypes" :supertype="supertype" />
          <CardGlare :subtypes="subtypes" :rarity="rarity" />
        </div>
      </button>
    </div>
  </div>
</template>

<script>
import CardShine from "./CardShine.vue";
import CardGlare from "./CardShine.vue";

const springR = { stiffness: 0.066, damping: 0.25 };
const springD = { stiffness: 0.033, damping: 0.45 };
let springRotate = spring({ x: 0, y: 0 }, springR);
let springGlare = spring({ x: 50, y: 50, o: 0 }, springR);
let springBackground = spring({ x: 50, y: 50 }, springR);
let springRotateDelta = spring({ x: 0, y: 0 }, springD);
let springTranslate = spring({ x: 0, y: 0 }, springD);
let springScale = spring(1, springD);

export default {
  methods: {
    interact(e) {
      if (showcaseRunning) {
        clearTimeout(showcaseTimerEnd);
        clearTimeout(showcaseTimerStart);
        clearInterval(showcaseInterval);
        showcaseRunning = false;
      }

      if (isVisible && $activeCard && $activeCard !== thisCard) {
        return (interacting = false);
      }

      interacting = true;

      if (e.type === "touchmove") {
        e.clientX = e.touches[0].clientX;
        e.clientY = e.touches[0].clientY;
      }

      const $el = e.target;
      const rect = $el.getBoundingClientRect(); // get element's current size/position
      const absolute = {
        x: e.clientX - rect.left, // get mouse position from left
        y: e.clientY - rect.top, // get mouse position from right
      };
      const percent = {
        x: round((100 / rect.width) * absolute.x),
        y: round((100 / rect.height) * absolute.y),
      };
      const center = {
        x: percent.x - 50,
        y: percent.y - 50,
      };

      springBackground.stiffness = springR.stiffness;
      springBackground.damping = springR.damping;
      springBackground.set({
        x: round(50 + percent.x / 4 - 12.5),
        y: round(50 + percent.y / 3 - 16.67),
      });
      springRotate.stiffness = springR.stiffness;
      springRotate.damping = springR.damping;
      springRotate.set({
        x: round(-(center.x / 3.5)),
        y: round(center.y / 2),
      });
      springGlare.stiffness = springR.stiffness;
      springGlare.damping = springR.damping;
      springGlare.set({
        x: percent.x,
        y: percent.y,
        o: 1,
      });
    },
    interactEnd(e, delay = 500) {
      setTimeout(function () {
        const snapStiff = 0.01;
        const snapDamp = 0.06;
        interacting = false;

        springRotate.stiffness = snapStiff;
        springRotate.damping = snapDamp;
        springRotate.set({ x: 0, y: 0 }, { soft: 1 });

        springGlare.stiffness = snapStiff;
        springGlare.damping = snapDamp;
        springGlare.set({ x: 50, y: 50, o: 0 }, { soft: 1 });

        springBackground.stiffness = snapStiff;
        springBackground.damping = snapDamp;
        springBackground.set({ x: 50, y: 50 }, { soft: 1 });
      }, delay);
    },
    activate(e) {
      if ($activeCard && $activeCard === thisCard) {
        $activeCard = undefined;
      } else {
        $activeCard = thisCard;
        resetBaseOrientation();
      }
    },
    deactivate(e) {
      this.interactEnd();
      $activeCard = undefined;
    },
    reposition(e) {
      clearTimeout(debounce);
      debounce = setTimeout(() => {
        if ($activeCard && $activeCard === thisCard) {
          this.setCenter();
        }
      }, 300);
    },
    setCenter() {
      const rect = thisCard.getBoundingClientRect(); // get element's size/position
      const view = document.documentElement; // get window/viewport size

      const delta = {
        x: round(view.clientWidth / 2 - rect.x - rect.width / 2),
        y: round(view.clientHeight / 2 - rect.y - rect.height / 2),
      };
      springTranslate.set({
        x: delta.x,
        y: delta.y,
      });
    },
    popover() {
      const rect = thisCard.getBoundingClientRect(); // get element's size/position
      let delay = 100;
      let scaleW = (window.innerWidth / rect.width) * 0.9;
      let scaleH = (window.innerHeight / rect.height) * 0.9;
      let scaleF = 1.75;
      this.setCenter();
      if (firstPop) {
        delay = 1000;
        springRotateDelta.set({
          x: 360,
          y: 0,
        });
      }
      firstPop = false;
      springScale.set(Math.min(scaleW, scaleH, scaleF));
      this.interactEnd(null, delay);
    },
    retreat() {
      springScale.set(1, { soft: true });
      springTranslate.set({ x: 0, y: 0 }, { soft: true });
      springRotateDelta.set({ x: 0, y: 0 }, { soft: true });
      this.interactEnd(null, 100);
    },
    reset() {
      this.interactEnd(null, 0);
      springScale.set(1, { hard: true });
      springTranslate.set({ x: 0, y: 0 }, { hard: true });
      springRotateDelta.set({ x: 0, y: 0 }, { hard: true });
      springRotate.set({ x: 0, y: 0 }, { hard: true });
    },
  },
  data() {
    return {
      back_img:
        "https://tcg.pokemon.com/assets/img/global/tcg-card-back-2x.jpg",
      gallery: false,
      img_base: this.img.startsWith("http")
        ? ""
        : "https://images.pokemontcg.io/",
      front_img: "",
      galaxyPosition: Math.floor(Math.random() * 1500),
      thisCard: undefined,
      rotator: undefined,
      debounce: undefined,
      active: false,
      interacting: false,
      firstPop: true,
      loading: true,
      isVisible: document.visibilityState === "visible",
      showcaseInterval: undefined,
      showcaseTimerStart: undefined,
      showcaseTimerEnd: undefined,
      showcaseRunning: true,
      galaxyPosition: Math.floor(Math.random() * 1500),
    };
  },
  computed: {
    styles() {
      return `--mx: ${this.springGlare.x}%;
		--my: ${this.springGlare.y}%;
		--tx: ${this.springTranslate.x}px;
		--ty: ${this.springTranslate.y}px;
		--s: ${this.springScale};
		--o: ${this.springGlare.o};
		--rx: ${this.springRotate.x + this.springRotateDelta.x}deg;
		--ry: ${this.springRotate.y + this.springRotateDelta.y}deg;
		--pos: ${this.springBackground.x}% ${this.springBackground.y}%;
		--posx: ${this.springBackground.x}%;
		--posy: ${this.springBackground.y}%;
		--hyp: ${clamp(
      Math.sqrt(
        (this.springGlare.y - 50) * (this.springGlare.y - 50) +
          (this.springGlare.x - 50) * (this.springGlare.x - 50)
      ) / 50,
      0,
      1
    )};
    --galaxybg: center ${this.galaxyPosition}px;`;
    },
  },
  props: {
    name: { type: String, required: true },
    img: { type: String, required: true },
    number: { type: String, required: true },
    supertype: { type: String, required: true },
    subtypes: { type: Array, required: true },
    rarity: { type: String, required: true },
    showcase: { type: Boolean, required: true },
  },
  components: { CardShine, CardGlare },
};
</script>

<style>
:root {
  --mx: 50%;
  --my: 50%;
  --s: 1;
  --o: 0;
  --tx: 0px;
  --ty: 0px;
  --rx: 0deg;
  --ry: 0deg;
  --pos: 50% 50%;
  --posx: 50%;
  --posy: 50%;
  --hyp: 0;
}

.card {
  --radius: 4.55% / 3.5%;
  --back: #004177;
  --glow: #69d1e9;
  z-index: calc(var(--s) * 100);
  transform: translate3d(0, 0, 0.1px);
  -webkit-transform: translate3d(0, 0, 0.1px);
  will-change: transform, visibility;
  transform-style: preserve-3d;
  -webkit-transform-style: preserve-3d;
}

.card.interacting {
  z-index: calc(var(--s) * 120);
}

.card.active .card__translater,
.card.active .card__rotator {
  touch-action: none;
}

.card__translater,
.card__rotator {
  display: grid;
  perspective: 600px;
  transform-origin: center;
  -webkit-transform-origin: center;
  will-change: transform;
}

.card__translater {
  width: auto;
  position: relative;
  transform: translate3d(var(--tx), var(--ty), 0) scale(var(--s));
  -webkit-transform: translate3d(var(--tx), var(--ty), 0) scale(var(--s));
}

.card__rotator {
  transform: rotateY(var(--rx)) rotateX(var(--ry));
  transform-style: preserve-3d;
  -webkit-transform: rotateY(var(--rx)) rotateX(var(--ry));
  -webkit-transform-style: preserve-3d;
  box-shadow: 0px 10px 20px -5px black;
  border-radius: var(--radius);
  outline: none;
  transition: box-shadow 0.4s ease, outline 0.2s ease;
}
button.card__rotator {
  appearance: none;
  -webkit-appearance: none;
  border: none;
  background: top;
  padding: 0;
}

.card.active .card__rotator {
  box-shadow: 0 0 10px 0px var(--glow), 0 0 10px 0px var(--glow),
    0 0 30px 0px var(--glow);
}

.card__rotator:focus {
  box-shadow: 0 0 10px 0px var(--glow), 0 0 10px 0px var(--glow),
    0 0 30px 0px var(--glow);
}

.card.active .card__rotator:focus {
  box-shadow: 0px 10px 30px 3px black;
}

.card__rotator :global(*) {
  width: 100%;
  display: grid;
  grid-area: 1/1;
  border-radius: var(--radius);
  image-rendering: optimizeQuality;
  transform-style: preserve-3d;
  -webkit-transform-style: preserve-3d;
}

.card__rotator img {
  outline: 1px solid transparent;
  aspect-ratio: 0.716;
  height: auto;
}

.card__back {
  background-color: var(--back);
  transform: rotateY(180deg) translateZ(1px);
  -webkit-transform: rotateY(180deg) translateZ(1px);
  backface-visibility: visible;
}

.card__front,
.card__front * {
  backface-visibility: hidden;
}

.card__front {
  opacity: 1;
  transition: opacity 0.33s ease-out;
}

.loading .card__front {
  opacity: 0;
}

.loading .card__back {
  transform: rotateY(0deg);
  -webkit-transform: rotateY(0deg);
}
</style>
