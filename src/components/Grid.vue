<template>
  <v-container>
    <v-row>
      <v-col cols="12" md="10" lg="8" class="mx-auto">
        <v-card>
          <v-card-title class="text-center">Eventos Cadastrados</v-card-title>
          <v-card-text>
            <v-list v-if="events.length">
              <v-list-item v-for="(event, index) in events" :key="index">
                <v-list-item-content>
                  <v-list-item-title>{{ event.name }}</v-list-item-title>
                  <v-list-item-subtitle>{{ formatDate(event.date) }}</v-list-item-subtitle>
                  <v-list-item-subtitle>{{ event.location }}</v-list-item-subtitle>
                  <v-list-item-subtitle>{{ event.description }}</v-list-item-subtitle>
                  <v-img v-if="event.image" :src="event.image" aspect-ratio="16/9" class="my-2"></v-img>
                </v-list-item-content>
                <v-list-item-action>
                  <v-btn color="primary" @click="openParticipationForm(event)">Participar</v-btn>
                  <v-btn color="secondary" @click="openMap(event.url)">Ver no Google Maps</v-btn>
                  <v-btn color="green" @click="openDialog(event.id)">Visualizar participantes</v-btn>
                </v-list-item-action>
              </v-list-item>
            </v-list>
            <div v-else class="text-center">
              <v-icon color="grey" size="64">mdi-emoticon-sad-outline</v-icon>
              <p>Nenhum evento cadastrado!</p>
            </div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <!-- Referência ao componente ParticipationForm com props -->
    <ParticipationForm v-if="selectedEvent" ref="participationForm" :event="selectedEvent" />
  </v-container>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import ParticipationForm from '../components/participantes.vue'

const events = ref([])
const selectedEvent = ref(null) // Para armazenar o evento selecionado
const participationForm = ref(null) // Adicionando a referência aqui

// Função para carregar eventos do Local Storage
const loadEvents = () => {
  const storedEvents = []
  const keys = Object.keys(localStorage)
  keys.forEach(key => {
    if (key.startsWith('event_')) {
      const event = JSON.parse(localStorage.getItem(key))
      storedEvents.push(event)
    }
  })
  events.value = storedEvents
}

// Carregar eventos ao montar o componente
onMounted(() => {
  loadEvents()
})

// Função para abrir o formulário de participação com o evento selecionado
const openParticipationForm = (event) => {
  selectedEvent.value = event // Armazena o evento selecionado
  if (participationForm.value) {
    participationForm.value.openDialog(event.id) // Passa o event.id, usando 'event.id' em vez de 'event.eventId'
  } else {
    console.error('ParticipationForm não está definido')
  }
}

// Função para abrir o Google Maps
const openMap = (url) => {
  if (url) {
    window.open(url, '_blank')
  }
}

// Função para formatar a data
const formatDate = (dateString) => {
  const date = new Date(dateString)
  const day = String(date.getDate()).padStart(2, '0')
  const month = String(date.getMonth() + 1).padStart(2, '0') // Janeiro é 0!
  const year = date.getFullYear()

  return `${day}/${month}/${year}`
}
</script>
