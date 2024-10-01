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
                  <v-btn color="green" @click="openParticipantsDialog(event.id)">Visualizar participantes</v-btn>
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

    <ParticipationForm v-if="selectedEvent" ref="participationForm" :event="selectedEvent" />
    <Visualiza v-if="selectedEventId" :eventId="selectedEventId" ref="visualizaDialog" />
  </v-container>
</template>
<script setup>
import { ref, defineExpose } from 'vue'

const dialog = ref(false)
const participants = ref([])

const openDialog = (eventId) => {
  // Lógica para carregar participantes baseados no eventId
  // Exemplo:
  const storedParticipants = JSON.parse(localStorage.getItem('participants')) || []
  participants.value = storedParticipants.filter(participant => participant.eventId === eventId)
  dialog.value = true
}

const closeDialog = () => {
  dialog.value = false
}

defineExpose({ openDialog, closeDialog })
</script>
