<script setup>
import { onMounted, ref } from "@vue/runtime-core";

defineProps({
  message: String,
  success: Boolean,
  error: Boolean,
});
defineEmits("closeModal");

const isHidden = ref(true);

onMounted(() => {
  setTimeout(() => {
    isHidden.value = false;
  }, 1);
});
</script>

<template>
  <div class="modal" :class="{ '-hidden': isHidden, '-success': success, '-error': error }">
    <div class="modal__text">
      {{ message }}
    </div>
    <button class="modal__button" @click="$emit('closeModal')">Close</button>
  </div>
</template>

<style lang="scss" scoped>
.modal {
  width: 450px;
  min-height: 175px;
  padding: 4rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-direction: column;
  border-radius: 15px;
  border: 0.5px solid;
  transition: all 0.5s ease-in-out;

  &.-success {
    border-color: var(--success-color);
    color: var(--success-color);

    button {
      color: #fff;
      border-color: var(--success-color);
      background-color: var(--success-color);
    }
  }

  &__text {
    font-size: 1.1rem;
    font-weight: 600;
  }

  &__button {
    width: 100%;
    font-weight: 600;
  }

  &.-error {
    border-color: var(--error-color);
    color: var(--error-color);

    button {
      color: #fff;
      border-color: var(--error-color);
      background-color: var(--error-color);
    }
  }

  &__text {
    font-size: 1.1rem;
    font-weight: 600;
  }

  &__button {
    width: 100%;
    font-weight: 600;
  }
}
</style>
