<template>
  <div
    class="sf-image"
    :class="{ 'sf-image--has-size': wrapper }"
    :style="wrapper"
    v-on="$listeners"
    @mouseover="overlay = true"
    @mouseleave="overlay = false"
  >
    <template v-if="typeof source === 'string'">
      <img
        v-if="show"
        ref="image"
        :src="source"
        :alt="alt"
        :width="width"
        :height="height"
      />
    </template>
    <template v-else>
      <picture>
        <source
          :srcset="source.desktop.url"
          :media="`(min-width: ${pictureBreakpoint}px)`"
        />
        <source
          :srcset="source.mobile.url"
          :media="`(max-width: ${pictureBreakpoint}px)`"
        />
        <img
          v-if="show"
          ref="image"
          :src="source"
          :alt="alt"
          :width="width"
          :height="height"
        />
      </picture>
    </template>
    <transition name="fade">
      <div v-if="showOverlay" class="sf-image__overlay">
        <slot />
      </div>
    </transition>
  </div>
</template>
<script>
import lozad from "lozad";
export default {
  name: "SfImage",
  props: {
    src: {
      type: [String, Object],
      default: () => ({ mobile: { url: "" }, desktop: { url: "" } }),
    },
    alt: {
      type: String,
      default: "",
    },
    width: {
      type: [String, Number],
      default: undefined,
    },
    height: {
      type: [String, Number],
      default: undefined,
    },
    lazy: {
      type: Boolean,
      default: true,
    },
    pictureBreakpoint: {
      type: Number,
      default: 1024,
    },
    rootMargin: {
      type: String,
      default: "",
    },
    threshold: {
      type: [String, Number],
      default: "",
    },
  },
  data() {
    return {
      show: false,
      overlay: false,
    };
  },
  computed: {
    source() {
      let src = this.src || "";
      if (typeof src === "object") {
        if (!src.desktop || !src.mobile) {
          const object = src.desktop || src.mobile || { url: "" };
          src = object.url;
        }
      }
      return src;
    },
    showOverlay() {
      return this.$slots.default && this.overlay;
    },
    wrapper() {
      return (
        this.width &&
        this.height &&
        `--_image-width: ${this.width}; --_image-height: ${this.height}`
      );
    },
  },
  watch: {
    lazy: {
      handler(value) {
        this.show = !value;
      },
      immediate: true,
    },
  },
  mounted() {
    if (!this.lazy) {
      this.show = true;
      return;
    }
    this.lozad();
  },
  methods: {
    lozad() {
      const vm = this;
      this.$nextTick(() => {
        const observer = lozad(vm.$el, {
          load() {
            vm.show = true;
          },
          rootMargin: this.rootMargin,
          threshold: this.threshold,
        });
        observer.observe();
      });
    },
  },
};
</script>
<style lang="scss">
@import "~@storefront-ui/shared/styles/components/atoms/SfImage.scss";
</style>
