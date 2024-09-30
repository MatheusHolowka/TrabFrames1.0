<template>
  <v-container>
    <v-row>
      <v-col v-for="(evento, index) in eventos" :key="index" cols="12" md="4">
        <v-card>
          <v-card-title>{{ evento.nome }}</v-card-title>
          <v-card-subtitle>{{ evento.data }}</v-card-subtitle>
          <v-card-text>{{ evento.descricao }}</v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref } from 'vue'

// Simulação de eventos cadastrados
const eventos = ref([
  { nome: 'Evento 1', data: '2024-09-16', descricao: 'Descrição do Evento 1' },
  { nome: 'Evento 2', data: '2024-09-17', descricao: 'Descrição do Evento 2' },
  { nome: 'Evento 3', data: '2024-09-18', descricao: 'Descrição do Evento 3' },
])
</script>
