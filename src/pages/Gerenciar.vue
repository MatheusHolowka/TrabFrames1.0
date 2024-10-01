<template>
  <v-container>
    <v-row>
      <v-col cols="12" md="10" lg="8" class="mx-auto">
        <v-card>
          <v-card-title class="text-center">Eventos Cadastrados</v-card-title>
          <v-card-text>
            <v-row v-if="events.length">
              <v-col cols="12" v-for="(event) in events" :key="event.eventId">
                <v-card class="mb-4">
                  <v-card-title>{{ event.name }}</v-card-title>
                  <v-card-subtitle>{{ event.date }}</v-card-subtitle>
                  <v-card-subtitle>{{ event.location }}</v-card-subtitle>
                  <v-card-text>{{ event.description }}</v-card-text>
                  <v-img
                    v-if="event.image"
                    :src="event.image"
                    aspect-ratio="16/9"
                    class="my-2"
                  ></v-img>
                  <v-card-actions>
                    <v-btn color="primary" class="mr-2" @click="viewParticipants(event.eventId)">Visualizar Participantes</v-btn>
                    <v-btn color="error" @click="confirmDelete(event.eventId)">Apagar</v-btn>
                  </v-card-actions>
                </v-card>
              </v-col>
            </v-row>
            <div v-else class="text-center">
              <v-icon color="grey" size="64">mdi-emoticon-sad-outline</v-icon>
              <p>Não possue nenhum evento Cadastrado!</p>
            </div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
    <ParticipantsDialog ref="participantsDialog" @deleteParticipant="deleteParticipant" @editParticipant="editParticipant" />
    <v-dialog v-model="confirmDialog" max-width="500">
      <v-card>
        <v-card-title class="headline">Confirmar Exclusão</v-card-title>
        <v-card-text>Você tem certeza que deseja apagar este evento?</v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="cancelDelete">Cancelar</v-btn>
          <v-btn color="red darken-1" text @click="deleteEvent">Apagar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import ParticipantsDialog from '../components/ParticipantesDialog.vue';

const events = ref([])
const confirmDialog = ref(false)
const eventIdToDelete = ref(null)

// Function to load events from local storage
const loadEvents = () => {
  const storedEvents = JSON.parse(localStorage.getItem('events')) || []
  events.value = storedEvents
}

// Load events when the component is mounted
onMounted(() => {
  loadEvents()
})

const participantsDialog = ref(null)

const viewParticipants = (eventId) => {
  participantsDialog.value.openDialog(eventId)
}

const confirmDelete = (eventId) => {
  eventIdToDelete.value = eventId
  confirmDialog.value = true
}

const cancelDelete = () => {
  eventIdToDelete.value = null
  confirmDialog.value = false
}

const deleteEvent = () => {
  // Remove the event from the events list
  events.value = events.value.filter(event => event.eventId !== eventIdToDelete.value)
  localStorage.setItem('events', JSON.stringify(events.value))

  // Remove participants associated with the event
  const participants = JSON.parse(localStorage.getItem('participants')) || []
  const updatedParticipants = participants.filter(participant => participant.eventId !== eventIdToDelete.value)
  localStorage.setItem('participants', JSON.stringify(updatedParticipants))

  confirmDialog.value = false
  eventIdToDelete.value = null
}

const deleteParticipant = (participantId) => {
  const participants = JSON.parse(localStorage.getItem('participants')) || []
  const updatedParticipants = participants.filter(participant => participant.participantId !== participantId)
  localStorage.setItem('participants', JSON.stringify(updatedParticipants))
  participantsDialog.value.refreshParticipants()
}

const editParticipant = (participantId, updatedData) => {
  const participants = JSON.parse(localStorage.getItem('participants')) || []
  const participantIndex = participants.findIndex(participant => participant.participantId === participantId)
  if (participantIndex !== -1) {
    participants[participantIndex] = { ...participants[participantIndex], ...updatedData }
    localStorage.setItem('participants', JSON.stringify(participants))
    participantsDialog.value.refreshParticipants()
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
