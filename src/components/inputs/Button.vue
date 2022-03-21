<template>
  <button @click="btnClick" :style="style" :class="btnClass">{{ btnTitle }}</button>
</template>

<script>
export default {
  /* Component name */
  name: "Button",

  /* Component's props */
  props: {
    btnTitle: {
      default: 'Click',
      type: String
    },
    btnColor: String,
    btnClass: String
  },

  /* Component's methods */
  methods: {
    btnClick(e) {
      const x = e.offsetX;
      const y = e.offsetY;

      const ripple = document.createElement('span');

      ripple.style.left = x + 'px';
      ripple.style.top = y + 'px';

      e.target.appendChild(ripple);

      setTimeout(() => ripple.remove(), 1000);
      
      this.$emit('btnClick', e)
    }
  },

  /* Component's computed property */
  computed: {
    style() {
      return {
        'background-color': this.btnColor
      }
    }
  }
}
</script>

<style lang="scss">
  @mixin square($size: null) {
    width: $size;
    height: $size;
  }

  @keyframes ripple {
    0% {
      opacity: .5;
      @include square(30px);
    }

    100% {
      opacity: 0;
      @include square(500px);
    }
  }

  button {
    border: none;
    outline: none;
    color: white;
    cursor: pointer;
    font-weight: 900;
    overflow: hidden;
    font-size: 1.1rem;
    padding: 1rem 2rem;
    position: relative;
    border-radius: .5rem;
    text-transform: uppercase;
    transition: .2s box-shadow;
    box-shadow: 0 1px 5px 0 rgba(black, .2);

    span {
      border-radius: 50%;
      position: absolute;
      pointer-events: none;
      background-color: white;
      animation: ripple 1s linear;
      transform: translate(-50%, -50%);
    }

    &:hover, &:focus {
      box-shadow: 0 5px 10px 0 rgba(black, .4);
    }
  }
</style>