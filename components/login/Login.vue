<template>
  <div class="flex flex-col justify-center items-center m-0 h-full">
    <div class="mt-16">
      <img src="~/assets/logo.png" class="w-80 my-2" alt="" />
    </div>
    <div class="my-4">
      <img src="~/assets/missao.png" class="w-56 my-2" alt="" />
    </div>
    <div>
      <p class="text-white font-medium text-3xl">Faça seu login</p>
    </div>

    <v-form>
      <v-container>
        <v-text-field
          v-model="login"
          dark
          placeholder="Login"
          prepend-inner-icon="mdi-account"
          outlined
          color="indigo darken-3"
        ></v-text-field>
        <v-text-field
          v-model="password"
          dark
          placeholder="Senha"
          prepend-inner-icon="mdi-lock"
          outlined
          :append-icon="show ? 'mdi-eye' : 'mdi-eye-off'"
          color="indigo darken-3"
          :type="show ? 'text' : 'password'"
          @click:append="show = !show"
        ></v-text-field>
        <v-row align="center" justify="space-around">
          <v-col class="flex justify-center">
            <v-btn large block elevation="4" @click="signin">
              <span v-if="loading">
                <i class="el-icon-loading"></i> Aguarde...
              </span>
              <span v-else class="text-[#1A237E] text-lg">Entrar</span>
            </v-btn>
          </v-col>
        </v-row>
      </v-container>
    </v-form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      show: false,
      login: '',
      password: '',
      loading: false,
    }
  },

  methods: {
    async signin() {
      try {
        this.loading = true
        await this.$auth.loginWith('local', {
          data: { login: this.login, password: this.password },
        })
        this.$router.push('/')
      } catch (error) {
        console.error(error)
        if (
          error.response &&
          error.response.data &&
          error.response.data.error
        ) {
          this.$toast.error(error.response.data.message, {
            position: 'top-center',
          })
        } else {
          this.$toast.error('Não foi possível conectar ao servidor.', {
            position: 'top-center',
          })
        }
      } finally {
        this.loading = false
      }
    },

    handleExit() {
      // Redirecionar o usuário, fechar um modal, etc.
      // Por exemplo:
      this.$router.push('/')
    },
  },
}
</script>

<style>
</style>
