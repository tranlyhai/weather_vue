<template>
  <Teleport to="body">
    <Transition name="modal-outer">
      <div
        class="absolute top-0 left-0 flex justify-center w-full h-screen px-8 bg-black bg-opacity-30"
        v-show="modalActive">
        <Transition name="modal-inner">
          <div class="self-start max-w-screen-md p-4 mt-32 bg-white"
          v-if="modalActive">
            <slot />
            <button type="button" class="px-6 py-2 mt-8 text-white bg-weather-primary" @click="$emit('close-modal')">
              Close
            </button>
          </div>
        </Transition>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup>
defineEmits(["close-modal"]);
defineProps({
  modalActive: {
    type: Boolean,
    default: false,
  }
});
</script>

<style scoped>
.modal-outer-enter-active,
.modal-outer-leave-active {
  transition: opacity 0.3s cubic-bezier(0.52, 0.02, 0.19, 1.02);
}

.modal-outer-enter-from,
.modal-outer-leave-to {
  opacity: 0;
}

.modal-inner-enter-active {
  transition: all 0.3s cubic-bezier(0.52, 0.02, 0.19, 1.02) 0.15s;
}

.modal-inner-leave-active {
  transition: all 0.3s cubic-bezier(0.52, 0.02, 0.19, 1.02);
}

.modal-inner-enter-from {
  opacity: 0;
  transform: scale(0.8);
}

.modal-inner-leave-to {
  transform: scale(0.8);
}
</style>
