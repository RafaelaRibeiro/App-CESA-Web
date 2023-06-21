<template>
  <div>
    <div
      class="keyboard-row"
      v-for="(row, index) in keyboardLayout"
      :key="index"
    >
      <v-btn
        color="primary"
        v-bind="size"
        v-for="key in row"
        :key="key"
        @click="handleKeyClick(key)"
        :class="{ space: key === 'Space', key: key !== 'Space' }"
        class="sm:m-0.5 md:m-1 w-12"
      >
        <span v-if="key === 'Caps Lock'"
          ><v-icon>mdi-keyboard-caps</v-icon></span
        >
        <span v-else-if="key === 'Backspace'"
          ><v-icon>mdi-backspace</v-icon></span
        >
        <span v-else-if="key === 'Space'"
          ><v-icon>mdi-keyboard-space</v-icon></span
        >
        <span v-else>{{ key }}</span>
      </v-btn>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      keyboardLayout: [
        ['1', '2', '3', '4', '5', '6', '7', '8', '9', '0', 'Backspace'],
        ['Q', 'W', 'E', 'R', 'T', 'Y', 'U', 'I', 'O', 'P'],
        ['A', 'S', 'D', 'F', 'G', 'H', 'J', 'K', 'L', 'Ç'],
        ['Z', 'X', 'C', 'V', 'B', 'N', 'M', ',', '.'],
        ['Space'],
      ],
    }
  },

  computed: {
    size() {
      const size = { xs: 'x-small', sm: 'small' }[this.$vuetify.breakpoint.name]
      return size ? { [size]: true } : {}
    },
  },

  methods: {
    handleKeyClick(key) {
      if (key === 'Backspace') {
        // Lógica para remover o último caractere
        this.$emit('back')
      } else if (key === 'Space') {
        // Lógica para adicionar espaço
        this.$emit('input', ' ')
      } else {
        // Lógica para lidar com outros botões
        this.$emit('input', key)
      }
    },
  },
}
</script>

<style scoped>
.keyboard-row {
  display: flex;
  justify-content: center;
}

/* button {
  padding: 5px;
  margin: 2px;
} */

.space {
  min-width: 20rem !important;
}

@media (min-width: 540px) {
  .key {
    min-width: 48px !important;
  }
}

@media (min-width: 768px) {
  .key {
    min-width: 64px !important;
  }
}
</style>
