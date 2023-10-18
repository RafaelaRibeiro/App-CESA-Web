<template>
  <div class="m-0 h-full">
    <div class="flex flex-col justify-center items-center">
      <loading :isLoading="isLoading" class="z-50" />

      <div class="mt-8">
        <img src="~/assets/logo.png" class="w-80 my-2" alt="" />
      </div>
      <div class="my-4">
        <img src="~/assets/missao.png" class="w-56 my-2" alt="" />
      </div>
      <div>
        <p class="text-white font-bold text-5xl">Seja Bem Vindo</p>
      </div>
      <div class="w-full mt-4 px-3"></div>

      <div class="container mx-auto p-6">
        <div
          class="flex flex-col gap-4 relative rounded-lg border-2 border-gray-300 h-20"
        >
          <div
            class="flex flex-col h-full px-2 pt-2 text-center"
            v-show="!isTyping"
          >
            <span class="text-gray-400 text-xl"
              >Digite seu CPF ou nome completo</span
            >
            <span class="text-gray-400 text-xl">
              para dar continuidade ao seu atendimento
            </span>
          </div>
          <div class="h-20 absolute inset-0 flex items-center">
            <input
              class="p-2 rounded-lg border-2 border-transparent placeholder-gray-400 text-white text-3xl w-full focus:outline-none"
              type="text"
              placeholder=""
              v-model="myNumber"
              @input="handleInput"
            />
          </div>
        </div>
      </div>

      <div class="my-2">
        <v-btn large class="mt-0.5" dark @click="getPatients" color="white"
          ><v-icon large color="#1A237E" left>mdi-magnify</v-icon
          ><span class="text-[#1A237E] text-lg">Buscar</span></v-btn
        >
      </div>
      <div class="my-4">
        <keyboard v-on:input="onNumberInput" v-on:back="removeChar" />
      </div>

      <div>
        <img src="~/assets/rotary.png" class="w-[400px]" alt="" />
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

          {{ selectedItems }}

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
    <div class="flex mx-3 items-center justify-end z-40 h-32" @click="logout">
      <v-icon x-large dark>mdi-exit-to-app</v-icon>
    </div>
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
      isTyping: false,
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
    handleInput() {
      this.isTyping = this.myNumber.length > 0
    },
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

        this.$toast.error(data.message, {
          position: 'top-center',
          bodyClassName: ['custom-class-1'],
        })
        console.log(`Erro: ${data.message}`)
        this.myNumber = []
        this.isTyping = false
      } finally {
        this.isLoading = false // Oculta o componente de carregamento
      }
    },
    async createServiceOrder() {
      if (this.selectedItems.length === 0) {
        this.$toast.error('Nenhum item selecionado', {
          position: 'top-center',
        })
        return
      }
      this.dialog = false

      this.isLoading = true

      const appointment = this.selectedItems.map((item) => ({
        AGM_MED: item.AGM_MED,
        AGM_LOC: item.AGM_LOC,
        AGM_HINI: item.AGM_HINI,
        AGM_EXT: item.AGM_EXT,
        AGM_PAC: item.AGM_PAC,
        AGM_TPSMK: item.AGM_TPSMK,
        AGM_SMK: item.AGM_SMK,
      }))

      console.log(appointment)
      try {
        await this.$axios.put('/appointments/update', appointment)
        console.log('Agendamento Atualizado')

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

        await this.$axios.post('/service-order', orders)
        this.selectedItems = []
        this.patient = null
        this.myNumber = null
        this.exibirModal()
      } catch (error) {
        if (
          error.response &&
          error.response.data &&
          error.response.data.message
        ) {
          console.log(`Erro: ${error.response.data.message}`)
        } else {
          console.log(error)
        }

        if (error.response && error.response.status === 400) {
          this.$toast.error('Erro ao atualizar o agendamento', {
            position: 'top-center',
          })
        } else {
          this.$toast.error('Erro ao criar a ordem de serviço', {
            position: 'top-center',
          })
        }
      } finally {
        this.isLoading = false // Oculta o componente de carregamento
      }
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
      }, 7000) // Fechar o modal após 7 segundos (7000 milissegundos)
    },

    async logout() {
      await this.$auth.logout()
      this.$router.push('/login')
    },
  },
}
</script>

<style>
.Vue-Toastification__toast-body.custom-class-1 {
  font-size: 24px;
}
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
