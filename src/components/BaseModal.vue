<template>
  <transition name="fade">
    <div class="modal"
         v-if="open"
         @click="close"
         @keydown.esc="close"
    >
      <slot></slot>
    </div>
  </transition>
</template>

<script>
export default {
  name: 'BaseModal',
  props: {
    open: {
      required: true,
      type: Boolean,
    },
  },

  watch: {
    open() {
      if (this.open) {
        document.body.style.overflow = 'hidden';
      } else {
        setTimeout(() => {
          document.body.style.overflow = 'auto';
        }, 300);
      }
    },
  },

  methods: {
    close(event) {
      if (event.target === document.querySelector('.modal')) {
        document.body.style.overflow = 'auto';
        this.$emit('update:open', false);
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 6;
  background-color: rgba(51, 51, 51, .3);
}
</style>
