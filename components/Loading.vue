<template>
  <div
    v-if="isLoading"
    class="fixed inset-0 flex items-center justify-center bg-opacity-50 bg-gray-900"
  >
    <div class="flex flex-col items-center space-x-2">
      <div
        class="loader animate-spin rounded-full h-8 w-8 border-t-2 border-b-2"
      ></div>
      <span class="text-2xl">Carregando...</span>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    isLoading: {
      type: Boolean,
      required: true,
    },
  },
}
</script>

<style>
.loader {
  border-top-color: transparent !important;
  border-right-color: transparent !important;
  animation-duration: 1s;
  animation-iteration-count: infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.animate-spin {
  animation-name: spin;
  animation-timing-function: linear;
}

.fixed {
  top: 0; /* Adicionei a propriedade top: 0 para corrigir a posição fixa */
}
</style>
