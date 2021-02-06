<template>
  <v-app dark>
    <v-app-bar fixed app>
      <v-toolbar-title v-text="title" />
      <v-spacer />
    </v-app-bar>
    <v-main>
      <v-container>
        <nuxt />
      </v-container>
    </v-main>
    <v-footer app class="d-flex justify-center" height="35">
      &copy; {{ year }} — Made with 💜 by Jeff Kong
    </v-footer>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      drawer: false,
      fixed: false,
      year: new Date().getFullYear(),
      title: 'Contacts',
    }
  },
}
</script>
