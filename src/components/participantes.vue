<template>
  <v-dialog v-model="dialog" max-width="600px">
    <v-card>
      <v-card-title class="text-center">Participar do Evento</v-card-title>
      <v-card-text>
        <v-form>
          <v-text-field
            v-model="participant.name"
            label="Nome"
            prepend-inner-icon="mdi-account"
            :rules="[nameRules]"
          />
          <v-text-field
            v-model="participant.phone"
            label="Telefone"
            prepend-inner-icon="mdi-phone"
            :rules="[phoneRules]"
          />
          <v-text-field
            v-model="participant.email"
            label="Email"
            prepend-inner-icon="mdi-email"
            :rules="[emailRules]"
          />
          <v-file-input
            v-model="participant.image"
            label="Imagem (opcional)"
            prepend-inner-icon="mdi-image"
            @change="previewImage"
          />
          <v-img v-if="imagePreview" :src="imagePreview" aspect-ratio="16/9" class="my-2"></v-img>
        </v-form>
      </v-card-text>
      <v-card-actions>
        <v-btn color="error" @click="closeDialog">Cancelar</v-btn>
        <v-spacer />
        <v-btn color="primary" :disabled="!isFormValid" @click="submitForm">Enviar</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script setup>
import { ref, computed, defineExpose, defineProps } from 'vue';

// Define as props recebidas do componente pai
const props = defineProps({
  event: {
    type: Object,
    required: true
  }
});

const dialog = ref(false);
const participant = ref({
  name: '',
  phone: '',
  email: '',
  image: null,
  eventId: null
});
const imagePreview = ref(null);

// Validações
const nameRules = (value) => value.length > 2 || 'Nome deve ter mais de 2 caracteres';
const phoneRules = (value) => /^\d{11}$/.test(value) || 'Telefone deve ter 11 dígitos';
const emailRules = (value) => /.+@.+\..+/.test(value) || 'Email inválido';

// Computed para verificar a validade do formulário
const isFormValid = computed(() => {
  return (
    nameRules(participant.value.name) === true &&
    phoneRules(participant.value.phone) === true &&
    emailRules(participant.value.email) === true
  );
});

// Fecha o diálogo
const closeDialog = () => {
  dialog.value = false;
};

// Envia o formulário e salva o participante
const submitForm = () => {
  const existingParticipants = JSON.parse(localStorage.getItem('participants')) || [];

  if (participant.value.eventId === null) {
    console.error('No event ID specified!');
    return;
  }

  if (participant.value.image && typeof participant.value.image !== 'string') {
    const reader = new FileReader();
    reader.onload = (e) => {
      participant.value.image = e.target.result;
      existingParticipants.push({ ...participant.value });
      localStorage.setItem('participants', JSON.stringify(existingParticipants));
      closeDialog();
    };
    reader.readAsDataURL(participant.value.image);
  } else {
    existingParticipants.push({ ...participant.value });
    localStorage.setItem('participants', JSON.stringify(existingParticipants));
    closeDialog();
  }
  console.log('Participante salvo:', participant.value);
};

// Abre o diálogo com o eventId
const openDialog = (eventId) => {
  participant.value.eventId = eventId;
  dialog.value = true;
};

// Pré-visualiza a imagem
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

// Expor a função openDialog para o componente pai
defineExpose({ openDialog });
</script>
