<template>
  <v-card max-width="725" class="mx-auto">
    <v-card-title>Cadastro de Eventos</v-card-title>
    <v-card-text>
      <v-form>
        <v-text-field
          ref="locationInput"
          v-model="localEvent.location"
          prepend-inner-icon="mdi-map-marker"
          variant="underlined"
          label="Local do Evento"
          @focus="initializeAutocomplete"
        />
        <v-text-field
          v-model="localEvent.name"
          prepend-inner-icon="mdi-calendar"
          variant="underlined"
          label="Nome do Evento"
        />
        <v-text-field
          v-model="localEvent.date"
          prepend-inner-icon="mdi-calendar-clock"
          variant="underlined"
          label="Data do Evento"
          type="date"
        />
        <v-textarea
          v-model="localEvent.description"
          prepend-inner-icon="mdi-text"
          variant="underlined"
          label="Descrição do Evento"
        />
        <v-file-input
          v-model="localEvent.image"
          prepend-inner-icon="mdi-image"
          variant="underlined"
          label="Imagem do Evento"
          @change="previewImage"
        />
        <v-img v-if="imagePreview" :src="imagePreview" aspect-ratio="16/9" class="my-2"></v-img>
      </v-form>
    </v-card-text>
    <v-card-actions>
      <v-btn color="error" @click="resetForm">Cancelar</v-btn>
      <v-spacer />
      <v-btn color="primary" :disabled="!isFormValid" @click="saveEvent">Salvar</v-btn>
    </v-card-actions>
  </v-card>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  event: {
    type: Object,
    default: () => ({})  // Fallback para evitar undefined
  }
})

const emit = defineEmits(['save-event'])

const localEvent = ref({
  ...props.event,
  eventId: props.event?.eventId || Date.now(),  // Verifique se eventId existe
  url: ''     // Adiciona a propriedade url
})

const imagePreview = ref(null)
let autocomplete = null

const locationInput = ref(null)  // Ref para o campo de localização

// Computed property to check if the form is valid
const isFormValid = computed(() =>
  ['name', 'date', 'location', 'description'].every(field => localEvent.value[field])
)

// Function to reset the form
const resetForm = () => {
  Object.assign(localEvent.value, {
    name: '',
    date: '',
    location: '',
    description: '',
    image: null,
    eventId: Date.now(),
    url: ''     // Reinicia a url
  })
  imagePreview.value = null
}

// Async function to save the event
const saveEvent = async () => {
  if (localEvent.value.image && typeof localEvent.value.image !== 'string') {
    const imageData = await readFileAsDataURL(localEvent.value.image)
    localEvent.value.image = imageData
  }

  // Salvar no Local Storage
  const eventToSave = {
    ...localEvent.value,
    formatted_address: localEvent.value.location // Supondo que você queira salvar como formatted_address
  }

  localStorage.setItem(`event_${localEvent.value.eventId}`, JSON.stringify(eventToSave))

  emit('save-event', eventToSave)
  resetForm()
}

// Function to preview image
const previewImage = (event) => {
  const file = event.target.files[0]
  if (file) {
    readFileAsDataURL(file).then(data => {
      imagePreview.value = data
    })
  } else {
    imagePreview.value = null
  }
}

// Helper function to read file as Data URL
const readFileAsDataURL = (file) => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.onload = () => resolve(reader.result)
    reader.onerror = reject
    reader.readAsDataURL(file)
  })
}

// Function to initialize Google Maps Autocomplete
const initializeAutocomplete = () => {
  if (!autocomplete) {
    const inputElement = locationInput.value.$el.querySelector('input')  // Obtém o input interno
    autocomplete = new google.maps.places.Autocomplete(inputElement)
    autocomplete.addListener('place_changed', handlePlaceSelect)
  }
}

// Function to handle place selection
const handlePlaceSelect = () => {
  const place = autocomplete.getPlace()
  if (place && place.geometry) {
    localEvent.value.location = place.formatted_address || ''
    localEvent.value.url = place.url || ''     // Salva a URL
    console.log('Selected place:', place)
  }
}
</script>
