<template>
  <v-dialog v-model="dialog" max-width="600px">
    <v-card>
      <v-card-title class="text-h5">Adicionar Participante</v-card-title>
      <v-card-text>
        <v-form>
          <v-text-field
            v-model="participant.name"
            :rules="[nameRules]"
            label="Nome"
            required
          ></v-text-field>
          <v-text-field
            v-model="participant.phone"
            :rules="[phoneRules]"
            label="Telefone"
            required
          ></v-text-field>
          <v-text-field
            v-model="participant.email"
            :rules="[emailRules]"
            label="Email"
            required
          ></v-text-field>
          <v-file-input
            v-model="participant.image"
            label="Imagem"
            @change="previewImage"
          ></v-file-input>
          <v-img v-if="imagePreview" :src="imagePreview" aspect-ratio="16/9" class="my-2"></v-img>
        </v-form>
      </v-card-text>
      <v-card-actions>
        <v-btn color="error" @click="closeDialog">Cancelar</v-btn>
        <v-spacer></v-spacer>
        <v-btn color="primary" :disabled="!isFormValid" @click="saveParticipant">Salvar</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script setup>
import { ref, computed } from 'vue';

const dialog = ref(false);
const participant = ref({
  name: '',
  phone: '',
  email: '',
  image: null,
  eventId: null, // Inicializa o eventId como null
});
const imagePreview = ref(null);

// Validações
const nameRules = (value) => value.length > 2 || 'Nome deve ter mais de 2 caracteres';
const phoneRules = (value) => /^\d{11}$/.test(value) || 'Telefone deve ter 11 dígitos';
const emailRules = (value) => /.+@.+\..+/.test(value) || 'Email inválido';
const idRules = (value) => value !== null || 'ID do evento é obrigatório';

// Computed para verificar a validade do formulário
const isFormValid = computed(() => {
  return (
    nameRules(participant.value.name) === true &&
    phoneRules(participant.value.phone) === true &&
    emailRules(participant.value.email) === true &&
    idRules(participant.value.eventId) === true
  );
});

// Fecha o diálogo e limpa os dados do participante
const closeDialog = () => {
  dialog.value = false;
  participant.value = {
    name: '',
    phone: '',
    email: '',
    image: null,
    eventId: null, // Reseta o eventId para null
  };
  imagePreview.value = null;
};

// Salva o participante
const saveParticipant = () => {
  const participants = JSON.parse(localStorage.getItem('participants')) || [];
  participants.push({ ...participant.value });
  localStorage.setItem('participants', JSON.stringify(participants));
  console.log('Participante salvo:', participant.value);
  closeDialog();
};

// Abre o diálogo com o eventId
const openDialog = (eventId) => {
  participant.value.eventId = eventId; // Define o eventId ao abrir o diálogo
  dialog.value = true;
};

// Pré-visualização da imagem
const previewImage = (event) => {
  const file = event.target.files[0];
  if (file) {
    const reader = new FileReader();
    reader.onload = (e) => {
      imagePreview.value = e.target.result;
    };
    reader.readAsDataURL(file);
  } else {
    imagePreview.value = null;
  }
};

defineExpose({ openDialog });
</script>
