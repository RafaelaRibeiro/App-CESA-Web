<template>
  <div>
    <v-checkbox
      v-model="selected"
      :value="appointment"
      @change="onCheckboxChange"
    >
      <template v-slot:label>
        <div>
          <div class="label-text mb-1">
            <span class="font-semibold text-gray-700">Procedimento:</span>
            <span class="sm:text-[14px] md:text-[16px]">
              {{ appointment.SMK.SMK_NOME }}
            </span>
          </div>
          <div class="label-text mb-1">
            <span class="font-semibold text-gray-700">Médico:</span>
            <span class="sm:text-[14px] md:text-[16px]">
              {{ appointment.PSV_AGM_AGM_MEDToPSV.PSV_NOME }}
            </span>
          </div>
          <div class="label-text">
            <span class="font-semibold text-gray-700"
              >Data do agendamento:</span
            >
            <span class="sm:text-[16px] md:text-[18px]">
              {{ formatDateTime(appointment.AGM_HINI) }}
            </span>
          </div>
        </div>
      </template>
    </v-checkbox>
  </div>
</template>

<script>
export default {
  props: ['appointment'],
  data() {
    return {
      selected: false,
    }
  },

  methods: {
    onCheckboxChange() {
      this.$emit('checkbox-updated', this.selected, this.appointment)
    },
    formatDate(date) {
      const options = { day: '2-digit', month: '2-digit', year: 'numeric' }
      return new Date(date).toLocaleDateString('pt-BR', options)
    },
    formatDateTime(dateTime) {
      const dateOptions = { day: '2-digit', month: '2-digit', year: 'numeric' }
      const timeOptions = { hour: 'numeric', minute: 'numeric' }

      const date = new Date(dateTime).toLocaleDateString('pt-BR', dateOptions)
      const time = new Date(dateTime).toLocaleTimeString('pt-BR', timeOptions)

      return `${date} ${time}`
    },
  },
}
</script>

<style></style>
