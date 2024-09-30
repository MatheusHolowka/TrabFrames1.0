<template>
  <form @submit.prevent="submitForm">
    <v-text-field
      v-model="state.name"
      :counter="10"
      :error-messages="v$.name.$errors.map(e => e.$message)"
      label="Name"
      required
      @blur="v$.name.$touch"
      @input="v$.name.$touch"
    ></v-text-field>

    <!-- Substituímos o campo de email por um campo de data com suporte para digitar -->
    <v-menu
      v-model="menu"
      :close-on-content-click="false"
      :nudge-right="40"
      transition="scale-transition"
      offset-y
      max-width="290px"
      min-width="auto"
    >
      <template v-slot:activator="{ on, attrs }">
        <v-text-field
          v-model="state.date"
          label="Data"
          prepend-icon="mdi-calendar"
          readonly
          v-bind="attrs"
          v-on="on"
          :error-messages="v$.date.$errors.map(e => e.$message)"
          required
        ></v-text-field>
      </template>
      <v-date-picker
        v-model="state.date"
        no-title
        scrollable
        @input="menu = false" <!-- Fecha o menu ao selecionar -->
      ></v-date-picker>
    </v-menu>

    <v-select
      v-model="state.select"
      :error-messages="v$.select.$errors.map(e => e.$message)"
      :items="items"
      label="Item"
      required
      @blur="v$.select.$touch"
      @change="v$.select.$touch"
    ></v-select>

    <v-checkbox
      v-model="state.checkbox"
      :error-messages="v$.checkbox.$errors.map(e => e.$message)"
      label="Do you agree?"
      required
      @blur="v$.checkbox.$touch"
      @change="v$.checkbox.$touch"
    ></v-checkbox>

    <v-btn class="me-4" @click="submitForm">
      submit
    </v-btn>
    <v-btn @click="clear">
      clear
    </v-btn>
  </form>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { useVuelidate } from '@vuelidate/core'
import { required } from '@vuelidate/validators'

const initialState = {
  name: '',
  date: '',
  select: null,
  checkbox: null,
}

const state = reactive({
  ...initialState,
})

const items = [
  'Item 1',
  'Item 2',
  'Item 3',
  'Item 4',
]

const rules = {
  name: { required },
  date: { required }, // Validação para a data
  select: { required },
  checkbox: { required },
}

const v$ = useVuelidate(rules, state)
const menu = ref(false) // Controla o estado do seletor de data

function submitForm() {
  v$.$touch() // Toca todos os campos para exibir erros
  if (v$.$pending || v$.$invalid) {
    return // Se ainda estiver validando ou inválido, não submeter
  }
  // Lógica de envio do formulário
  console.log('Formulário válido, enviando dados:', state)
}

function clear() {
  v$.value.$reset()

  for (const [key, value] of Object.entries(initialState)) {
    state[key] = value
  }
}
</script>
