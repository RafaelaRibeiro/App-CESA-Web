<template>
  <v-app dark>
    <div
      class="h-screen w-full bg-[url('~/assets/Fundo.png')] bg-cover bg-center"
    >
      <!-- class="h-screen w-full bg-[url('~/assets/Fundo.png')] bg-cover bg-center" -->
      <main class="h-full">
        <Nuxt />
      </main>
    </div>

    <!-- <v-footer :absolute="!fixed" app>
      <span>&copy; {{ new Date().getFullYear() }}</span>
    </v-footer> -->
  </v-app>
</template>

<script>
export default {
  name: 'DefaultLayout',
  data() {
    return {
      clipped: false,
      drawer: false,
      fixed: false,
      items: [
        {
          icon: 'mdi-apps',
          title: 'Welcome',
          to: '/',
        },
        {
          icon: 'mdi-chart-bubble',
          title: 'Inspire',
          to: '/inspire',
        },
      ],
      miniVariant: false,
      right: true,
      rightDrawer: false,
      title: 'Vuetify.js',
    }
  },
}
</script>
