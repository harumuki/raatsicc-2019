<template>
  <figure ref="placeholder" class="image" :unmask="unmask" :class="figureClass">
    <svg class="u-hidden" xmlns="http://www.w3.org/2000/svg" version="1.1">
      <defs>
        <filter id="blur" color-interpolation-filters="sRGB">
          <feGaussianBlur stdDeviation="0" in="SourceGraphic"></feGaussianBlur>
        </filter>
      </defs>
    </svg>
    <!-- the low-res image has no alt tag to avoid -->
    <!-- the shitty broken image icon when loading -->
    <img :class="['low-rez', { 'is-loaded': largeUrl }]" :src="smallUrl" />
    <img
      :class="['high-rez', { 'is-loaded': largeUrl }]"
      :src="largeUrl"
      :alt="alt"
    />
  </figure>
</template>

<script>
/* cloudinary utils */
const cldBaseUrl = "https://res.cloudinary.com/pw-img-cdn/image/fetch";
const cldBlurUrl = `${cldBaseUrl}/w_50,e_blur:500`;

function calcImageDimension(length, pixelRatio) {
  return 100 * Math.round((length * pixelRatio) / 100);
}

export default {
  name: "LoadingImage",
  props: {
    src: {
      type: String,
      required: true
    },
    alt: {
      type: String,
      default: "this is an image"
    },
    unmask: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      // Set the smallUrl as an empty string to avoid
      // the image outline that show if there is no src attr
      smallUrl: "",
      largeUrl: undefined
    };
  },
  computed: {
    figureClass() {
      return this.unmask && "no-mask";
    }
  },
  mounted() {
    /* Get the image reference */
    const image = this.$refs.placeholder;

    /* Load a small image from cloudinary */
    const smallUrl = `${cldBlurUrl}/${this.src}`;
    var smallImage = new Image();
    smallImage.src = smallUrl;

    smallImage.onload = () => {
      this.smallUrl = smallUrl;
    };

    /* Load a large image from cloudinary */
    const { clientWidth } = image;
    // const pixelRatio = window.devicePixelRatio || 1.0;
    const pixelRatio = 1.333;
    const imageWidth = calcImageDimension(clientWidth, pixelRatio);
    const imageParams = `w_${imageWidth},c_fill,g_center,f_auto,q_auto`;
    const largeUrl = `${cldBaseUrl}/${imageParams}/${this.src}`;
    const largeImage = new Image();
    largeImage.src = largeUrl;

    largeImage.onload = () => {
      this.largeUrl = largeUrl;
    };
  }
};
</script>

<style scoped>
@import "../assets/styles/variables.css";

/* THEME */

.image {
  /* background-color: var(--c-white); */
  /* border: 1px solid var(--c-text); */
  /* box-shadow: var(--c-text) 0 0 0 1px; */
}

/* COMPONENT */

.image {
  margin-top: 0;
  overflow: hidden;
  position: relative;
}

.no-mask {
  overflow: initial;
}

.image img {
  min-height: 100%;
  min-width: 100%;
  transition: opacity 750ms ease;
  width: 100%;

  /* hide the alt text */
  /* https://stackoverflow.com/questions/36305805/how-to-hide-alt-text-using-css-when-the-image-is-not-present */
  /* text-indent: -9999px; */
  text-indent: 100%;
  white-space: nowrap;
  overflow: hidden;
}

/*
  1. enable Safari to keep sharp edges
     …but only works if pos-abs within an aspect-ratio div.
*/
.low-rez {
  /* filter: blur(50px); */
  filter: url("#blur");
  transform: scale(1); /*1*/
}

.high-rez {
  left: 0;
  opacity: 0;
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
}

/*
  on full-res image load
*/

.low-rez.is-loaded {
  opacity: 0;
}

.high-rez.is-loaded {
  opacity: 1;
}
</style>
