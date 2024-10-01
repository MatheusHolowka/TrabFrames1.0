<template>
  <v-dialog v-model="dialog" max-width="600px">
    <v-card>
      <v-card-title class="text-center">Participantes</v-card-title>
      <v-card-text>
        <!-- Mostrar participantes para o evento selecionado -->
        <v-list v-if="participants.length">
          <v-list-item v-for="(participant, index) in participants" :key="index">
            <v-list-item-content>
              <v-list-item-title>{{ participant.name }}</v-list-item-title>
              <v-list-item-subtitle>{{ participant.phone }}</v-list-item-subtitle>
              <v-list-item-subtitle>{{ participant.email }}</v-list-item-subtitle>
              <v-img
                v-if="participant.image"
                :src="participant.image"
                aspect-ratio="16/9"
                class="my-2"
              ></v-img>
            </v-list-item-content>
            <v-list-item-action>
              <v-btn icon @click="editParticipant(index)">
                <v-icon>mdi-pencil</v-icon>
              </v-btn>
              <v-btn icon @click="confirmDelete(index)">
                <v-icon>mdi-delete</v-icon>
              </v-btn>
            </v-list-item-action>
          </v-list-item>
        </v-list>
        <!-- Caso não haja participantes -->
        <div v-else class="text-center">
          <v-icon color="grey" size="64">mdi-emoticon-sad-outline</v-icon>
          <p>Nenhum participante cadastrado!</p>
        </div>
      </v-card-text>
      <v-card-actions>
        <v-btn color="primary" @click="closeDialog">Fechar</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>

  <!-- Diálogo de Edição -->
  <v-dialog v-model="editDialog" max-width="500">
    <v-card>
      <v-card-title class="headline">Editar Participante</v-card-title>
      <v-card-text>
        <v-form>
          <v-text-field v-model="editParticipantData.name" label="Nome" />
          <v-text-field v-model="editParticipantData.phone" label="Telefone" />
          <v-text-field v-model="editParticipantData.email" label="Email" />
          <v-file-input v-model="editParticipantData.image" label="Imagem (opcional)" @change="previewEditImage" />
          <v-img v-if="editImagePreview" :src="editImagePreview" aspect-ratio="16/9" class="my-2"></v-img>
        </v-form>
      </v-card-text>
      <v-card-actions>
        <v-btn color="blue darken-1" text @click="cancelEdit">Cancelar</v-btn>
        <v-btn color="green darken-1" text @click="saveEdit">Salvar</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>

  <!-- Diálogo de Confirmação de Exclusão -->
  <v-dialog v-model="confirmDialog" max-width="500">
    <v-card>
      <v-card-title class="headline">Confirmar Exclusão</v-card-title>
      <v-card-text>Você tem certeza que deseja remover este participante?</v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="blue darken-1" text @click="cancelDelete">Cancelar</v-btn>
        <v-btn color="red darken-1" text @click="deleteParticipant">Remover</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script setup>
import { ref, defineExpose } from 'vue'

const dialog = ref(false)
const participants = ref([])
const editDialog = ref(false)
const confirmDialog = ref(false)
const editParticipantData = ref({})
const editImagePreview = ref(null)
const participantIndexToEdit = ref(null)
const participantIndexToDelete = ref(null)

// Função para abrir o diálogo e filtrar os participantes com base no eventId
const openDialog = (eventId) => {
  console.log('eventId recebido:', eventId); // Log para verificar o ID recebido
  const storedParticipants = JSON.parse(localStorage.getItem('participants')) || []

  // Filtrando participantes com base no eventId
  participants.value = storedParticipants.filter(participant => participant.eventId === eventId)

  console.log('Participantes filtrados:', participants.value); // Log dos participantes filtrados
  dialog.value = true
}

// Função para fechar o diálogo
const closeDialog = () => {
  dialog.value = false
}

// Função para abrir o diálogo de edição
const editParticipant = (index) => {
  participantIndexToEdit.value = index
  editParticipantData.value = { ...participants.value[index] }
  editImagePreview.value = participants.value[index].image
  editDialog.value = true
}

// Função para cancelar a edição
const cancelEdit = () => {
  editDialog.value = false
  editParticipantData.value = {}
  editImagePreview.value = null
}

// Função para salvar a edição
const saveEdit = () => {
  if (editParticipantData.value.image && typeof editParticipantData.value.image !== 'string') {
    const reader = new FileReader()
    reader.onload = (e) => {
      editParticipantData.value.image = e.target.result
      participants.value[participantIndexToEdit.value] = { ...editParticipantData.value }
      updateLocalStorage()
      editDialog.value = false
    }
    reader.readAsDataURL(editParticipantData.value.image)
  } else {
    participants.value[participantIndexToEdit.value] = { ...editParticipantData.value }
    updateLocalStorage()
    editDialog.value = false
  }
}

// Função para pré-visualizar a imagem editada
const previewEditImage = (event) => {
  const file = event.target.files[0]
  if (file) {
    const reader = new FileReader()
    reader.onload = (e) => {
      editImagePreview.value = e.target.result
    }
    reader.readAsDataURL(file)
  } else {
    editImagePreview.value = null
  }
}

// Função para abrir o diálogo de confirmação de exclusão
const confirmDelete = (index) => {
  participantIndexToDelete.value = index
  confirmDialog.value = true
}

// Função para cancelar a exclusão
const cancelDelete = () => {
  participantIndexToDelete.value = null
  confirmDialog.value = false
}

// Função para remover o participante
const deleteParticipant = () => {
  if (participantIndexToDelete.value !== null) {
    const participantIdToDelete = participants.value[participantIndexToDelete.value].participantId
    participants.value.splice(participantIndexToDelete.value, 1)
    updateLocalStorage(participantIdToDelete)
    confirmDialog.value = false
    participantIndexToDelete.value = null
  }
}

// Função para atualizar o localStorage
const updateLocalStorage = (participantIdToDelete) => {
  const storedParticipants = JSON.parse(localStorage.getItem('participants')) || []
  const updatedParticipants = storedParticipants.filter(participant => participant.participantId !== participantIdToDelete)
  localStorage.setItem('participants', JSON.stringify(updatedParticipants))
}

// Função para carregar os participantes do localStorage
const loadParticipants = () => {
  participants.value = JSON.parse(localStorage.getItem('participants')) || []
}

// Carregar os participantes quando o componente é montado
onMounted(() => {
  loadParticipants()
})
// Expondo a função openDialog para que ela possa ser chamada externamente
defineExpose({ openDialog })
</script>
