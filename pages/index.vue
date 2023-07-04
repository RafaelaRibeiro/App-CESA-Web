<template>
  <div class="flex flex-col justify-center items-center m-0">
    <loading :isLoading="isLoading" class="z-50" />
    <div>
      <img
        src="~/assets/logo vertical_page-0001.jpg"
        class="w-64 my-2"
        alt=""
      />
    </div>
    <div class="w-4/5 flex">
      <v-text-field
        outlined
        dense
        placeholder="
          Digite seu CPF ou seu nome completo"
        v-model="myNumber"
        clearable
        class="mr-3"
      >
      </v-text-field>

      <v-btn class="mt-0.5" dark @click="getPatients" color="#1A237E"
        >Buscar</v-btn
      >
      <!-- <input v-mask="'###.###.###-##'" v-model="myNumber" /> -->
    </div>
    <div>
      <keyboard v-on:input="onNumberInput" v-on:back="removeChar" />
    </div>
    <div class="mt-3 flex justify-center items-center"></div>
    <div>
      <img src="~/assets/CASA_LOGO ROTARY.jpg" class="w-72 mt-2" alt="" />
    </div>

    <v-dialog v-model="dialog" transition="dialog-transition">
      <div v-if="!patient">
        <div
          class="text-2xl bg-slate-300 p-3 text-gray-700 flex justify-between sticky top-0 z-10"
        >
          <div>Selecione o paciente</div>
          <div>
            <v-btn @click="dialog = false" color="error">Cancelar</v-btn>
          </div>
        </div>

        <v-data-table
          :headers="headers"
          :items="patients"
          class="elevation-1"
          hide-default-footer
          @click:row="openDetails"
          :items-per-page="patients.length"
        >
          <!-- eslint-disable-next-line vue/valid-v-slot -->
          <template v-slot:item.PAC_NOME="{ item }">
            <span class="sm:text-[12px] md:text-[14px]">
              {{ item.PAC_NOME }}
            </span>
          </template>
          <!-- eslint-disable-next-line vue/valid-v-slot -->
          <template v-slot:item.PAC_NOME_MAE="{ item }">
            <span class="sm:text-[12px] md:text-[14px]">
              {{ item.PAC_NOME_MAE }}
            </span>
          </template>
          <!-- eslint-disable-next-line vue/valid-v-slot -->
          <template v-slot:item.PAC_NASC="{ item }">
            <span class="sm:text-[12px] md:text-[14px]">
              {{ formatDate(item.PAC_NASC) }}
            </span>
          </template>
        </v-data-table>
      </div>
      <div class="bg-white m-0 px-3" v-else>
        <div class="mb-4 pt-5">
          <span class="font-semibold text-gray-700 text-lg">
            Olá, {{ trimEndPacNome }}!
          </span>
          <br />
          Selecione seus agendamentos e clique em confirmar
        </div>
        <div class="mb-1">
          <div v-for="appointment in appointments" :key="appointment.PAG_REG">
            <appointment-item
              :appointment="appointment"
              @checkbox-updated="updateSelectedItems"
            />
          </div>
        </div>

        <div class="flex justify-end p-3">
          <v-btn dark color="#1A237E" @click="createServiceOrder" class="mr-2"
            >Confirmar</v-btn
          >
          <v-btn @click="patient = null" color="error">Voltar</v-btn>
        </div>
      </div>
    </v-dialog>
    <v-dialog v-model="showModal" max-width="500px ">
      <!-- Conteúdo do modal -->
      <div class="flex items-center justify-center">
        <div class="flex flex-col items-center bg-white rounded p-6">
          <!-- Ícone do Material Design Icons -->
          <span class="text-7xl text-green-500 mb-4"
            ><i class="mdi mdi-check-circle-outline"></i
          ></span>
          <h2 class="text-xl font-bold text-gray-700 mb-2">
            A ordem de serviço foi criada com sucesso.
          </h2>
          <p class="text-gray-700 text-center">
            Aguarde na sala de espera que você já será atendido.
          </p>
          <div class="w-full bg-gray-200 rounded h-2 mt-4">
            <div
              class="bg-green-500 rounded h-2"
              :class="{ 'animate-progress': showModal }"
            ></div>
          </div>
        </div>
      </div>
    </v-dialog>
  </div>
</template>

<script>
import Keyboard from '~/components/Keyboard.vue'
import AppointmentItem from '../components/appointments/AppointmentItem.vue'
import Loading from '~/components/Loading.vue'

export default {
  components: {
    Keyboard,
    AppointmentItem,
    Loading,
  },
  data() {
    return {
      isLoading: false,
      myNumber: '',
      patients: [],
      patient: null,
      appointments: [],
      selectedItems: [],
      dialog: false,
      showModal: false,
      dialogError: false,
      headers: [
        {
          text: 'Paciente',
          align: 'start',
          value: 'PAC_NOME',
          sortable: true,
          width: '25%',
        },
        {
          text: 'Nome da Mãe',
          align: 'start',
          value: 'PAC_NOME_MAE',
          sortable: true,
          width: '20%',
        },
        {
          text: 'Data de Nascimento',
          align: 'start',
          value: 'PAC_NASC',
          sortable: true,
          width: '20%',
        },
      ],
    }
  },

  computed: {
    trimEndPacNome() {
      if (this.patient && typeof this.patient.PAC_NOME === 'string') {
        return this.patient.PAC_NOME.trimEnd() // Remove espaços em branco no final do valor
      }
      return this.patient ? this.patient.PAC_NOME : ''
    },
  },

  methods: {
    async openDetails(value) {
      let appointments = await this.$axios.get(`/appointments/${value.PAC_REG}`)

      this.appointments = appointments.data

      let patient = await this.$axios.get(`/patients/${value.PAC_REG}`)

      this.patient = patient.data
    },
    formatDate(date) {
      const options = { day: '2-digit', month: '2-digit', year: 'numeric' }
      return new Date(date).toLocaleDateString('pt-BR', options)
    },

    onNumberInput(number) {
      this.myNumber += number
    },

    removeChar() {
      this.myNumber = this.myNumber.slice(0, -1)
    },

    async getPatients() {
      try {
        this.isLoading = true
        const patients = await this.$axios.get('/patients', {
          params: {
            search: this.myNumber,
          },
        })

        this.patients = patients.data
        this.dialog = true
      } catch (error) {
        const { data } = error.response
        this.$toast.error('Erro na busca dos dados', {
          position: 'top-center',
        })
        console.log(data.message)
      } finally {
        this.isLoading = false // Oculta o componente de carregamento
      }
    },
    async createServiceOrder() {
      if (this.selectedItems.length === 0) {
        // Caso nenhum item tenha sido selecionado
        this.$toast.error('Nenhum item selecionado', {
          position: 'top-center',
        })
        return
      }
      this.dialog = false

      this.isLoading = true

      const orders = this.selectedItems.map((item, index) => ({
        pac_reg: item.AGM_PAC,
        smmTpcod: item.AGM_TPSMK,
        smmCod: item.AGM_SMK,
        smmHonSeq: item.AGM_HON_SEQ,
        osmCnv: item.AGM_CNV_COD,
        smmVlr: item.AGM_VALOR,
        smmMed: item.AGM_MED,
        smmTab: item.CNV_TAB,
        smmNum: index + 1,
        agmHini: item.AGM_HINI,
      }))

      await this.$axios
        .post('/serviceOrder/create', orders)
        .then(() => {
          this.selectedItems = []
          this.patient = null
          this.myNumber = ''
          this.exibirModal()
        })
        .catch((error) => {
          this.$toast.error('Erro ao criar a ordem de serviço', {
            position: 'top-center',
          })
          console.log(error)
        })
        .finally(() => {
          this.isLoading = false // Oculta o componente de carregamento
        })
    },

    updateSelectedItems(selected, appointment) {
      if (selected) {
        this.selectedItems.push(appointment)
      } else {
        const index = this.selectedItems.findIndex(
          (item) => item === appointment
        )
        if (index !== -1) {
          this.selectedItems.splice(index, 1)
        }
      }
    },

    exibirModal() {
      this.showModal = true

      setTimeout(() => {
        this.showModal = false
      }, 7000) // Fechar o modal após 5 segundos (5000 milissegundos)
    },
  },
}
</script>

<style>
@keyframes progressAnimation {
  0% {
    width: 0;
  }
  100% {
    width: 100%;
  }
}

.animate-progress {
  animation-name: progressAnimation;
  animation-duration: 7s; /* Duração da animação (mesmo valor utilizado no setTimeout) */
  animation-timing-function: linear;
}

.v-dialog__content {
  align-items: start !important;
  margin-top: 20vh;
}
</style>
