<template>
  <v-app id="inspire">
    <v-navigation-drawer v-model="drawer">
      <Menu @logout="deslogar" titulo="Menu principal" /> <!--Filho = Menu, comunica com ele com props-->
    </v-navigation-drawer>
    <v-app-bar>
      <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>

      <v-app-bar-title>Eventos</v-app-bar-title>
    </v-app-bar>

    <v-main>
      <v-container>
        <RouterView />
        <Grid v-if="isHomeRoute" />
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useRoute } from 'vue-router'  // Importa o hook para acessar a rota atual


const drawer = ref(true)
const status = ref('Logado')
const deslogar = () => {
  status.value = 'Deslogado'
}

// Acessa a rota atual usando useRoute
const route = useRoute()

// Computa se a rota atual é a rota "/"
const isHomeRoute = computed(() => route.path === '/')
</script>
