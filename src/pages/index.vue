<template>
  <v-container>
    <v-row>
      <v-col cols="12" md="10" lg="8" class="mx-auto">
        <v-card>
          <v-card-title class="text-center">Eventos</v-card-title>
          <v-card-text>
            <v-row v-if="events.length">
              <v-col cols="12" v-for="(event, index) in events" :key="index">
                <v-card class="mb-4">
                  <v-card-title>{{ event.name }}</v-card-title>
                  <v-card-subtitle>{{ event.date }}</v-card-subtitle>
                  <v-card-subtitle>{{ event.location }}</v-card-subtitle>
                  <v-card-text>{{ event.description }}</v-card-text>
                  <v-img v-if="event.image" :src="event.image" aspect-ratio="16/9" class="my-2"></v-img>
                  <v-card-actions>
                    <v-btn color="primary" @click="openParticipationForm(event)">Participar</v-btn>
                    <v-btn color="secondary" @click="openMaps(event)">Ver no Maps</v-btn>
                  </v-card-actions>
                </v-card>
              </v-col>
            </v-row>
            <div v-else class="text-center">
              <v-icon color="grey" size="64">mdi-emoticon-sad-outline</v-icon>
              <p>Nenhum evento cadastrado!</p>
            </div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <!-- Referência ao componente ParticipationForm com props -->
    <ParticipationForm ref="participationForm" :event="selectedEvent" />
  </v-container>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import ParticipationForm from '../components/FormularioParticipantes.vue'

const events = ref([])
const selectedEvent = ref(null) // Para armazenar o evento selecionado

// Função para carregar eventos do localStorage
const loadEvents = () => {
  const storedEvents = JSON.parse(localStorage.getItem('events')) || []
  console.log('Eventos carregados do localStorage:', storedEvents) // Log para depuração
  events.value = storedEvents
}

// Carregar eventos quando o componente for montado
onMounted(() => {
  loadEvents()
})

const participationForm = ref(null)

// Função para abrir o formulário de participação com o evento selecionado
const openParticipationForm = (event) => {
  console.log('Evento selecionado:', event)
  selectedEvent.value = event // Armazena o evento selecionado
  participationForm.value.openDialog(event.eventId) // Passa o eventId
}

// Função para abrir o Google Maps com a URL da localização
const openMaps = (event) => {
  console.log('Evento para abrir no Maps:', event) // Log para depuração
  if (event && event.url) {
    window.open(event.url, '_blank')
  } else {
    alert('URL de localização não disponível')
  }
}
</script>

<style scoped>
.text-center {
  text-align: center;
}
.justify-center {
  justify-content: center;
}
.mb-4 {
  margin-bottom: 1rem;
}
</style>
