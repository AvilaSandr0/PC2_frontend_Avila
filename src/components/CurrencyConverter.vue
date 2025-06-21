<template>
  <q-card class="q-pa-xl q-mx-auto shadow-2" style="max-width: 400px">
    <q-card-section class="q-pb-none flex flex-center column">
      <q-icon name="trending_up" color="primary" size="36px" class="q-mb-xs" />
      <div class="text-h6 text-weight-bold">Conversión de Monedas</div>
    </q-card-section>
    <q-form @submit.prevent="convertCurrency">
      <q-card-section class="q-gutter-md">
        <q-input
          v-model.number="amount"
          type="number"
          label="Monto a convertir"
          :rules="[(val) => val > 0 || 'Ingrese un monto válido']"
          dense
          outlined
          required
        />
        <div class="row items-center q-gutter-sm">
          <div class="col-5">
            <q-select
              v-model="from"
              :options="[{ label: 'Seleccione moneda', value: '' }, ...currencyOptions]"
              label="Desde"
              dense
              outlined
              emit-value
              map-options
              :loading="loading"
              required
              :error="errorFrom"
              :error-message="errorFrom ? 'Debe seleccionar ambas monedas a cambiar' : ''"
            />
          </div>
          <div class="col-2 flex flex-center">
            <q-btn
              round
              color="white"
              text-color="primary"
              icon="sync_alt"
              @click="swapCurrencies"
              :disable="loading"
              size="md"
              flat
              unelevated
              class="swap-btn"
            />
          </div>
          <div class="col-5">
            <q-select
              v-model="to"
              :options="[{ label: 'Seleccione moneda', value: '' }, ...currencyOptions]"
              label="Hacia"
              dense
              outlined
              emit-value
              map-options
              :loading="loading"
              required
              :error="errorTo"
              :error-message="errorTo ? 'Debe seleccionar ambas monedas a cambiar' : ''"
            />
          </div>
        </div>
        <q-btn
          type="submit"
          label="Convertir"
          class="full-width gradient-btn q-mt-md"
          :loading="loading"
          unelevated
        />
      </q-card-section>
    </q-form>
    <q-card-section v-if="result !== null">
      <div class="result-box text-center">
        <span class="text-h6 text-weight-bold"
          >{{ amount }} {{ from }} equivalen a {{ result }} {{ to }}</span
        >
      </div>
    </q-card-section>
    <q-card-section v-if="error">
      <q-banner dense class="bg-red-2 text-red-10">{{ error }}</q-banner>
    </q-card-section>
  </q-card>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const amount = ref(1)
const from = ref('USD')
const to = ref('EUR')
const currencyOptions = ref([])
const result = ref(null)
const error = ref('')
const loading = ref(false)
const errorFrom = ref(false)
const errorTo = ref(false)

onMounted(async () => {
  loading.value = true
  try {
    const { data } = await axios.get('https://api.frankfurter.app/currencies')
    currencyOptions.value = Object.entries(data).map(([code, name]) => ({
      label: `${code} - ${name}`,
      value: code,
    }))
    // Set default currencies if available
    if (!currencyOptions.value.find((opt) => opt.value === from.value))
      from.value = currencyOptions.value[0].value
    if (!currencyOptions.value.find((opt) => opt.value === to.value))
      to.value = currencyOptions.value[1].value
  } catch {
    error.value = 'No se pudieron cargar las monedas.'
  } finally {
    loading.value = false
  }
})

const swapCurrencies = () => {
  const temp = from.value
  from.value = to.value
  to.value = temp
  result.value = null
}

const convertCurrency = async () => {
  error.value = ''
  result.value = null
  errorFrom.value = !from.value
  errorTo.value = !to.value
  // Validaciones
  if (!amount.value || amount.value <= 0) {
    error.value = 'Ingrese un monto válido.'
    return
  }
  if (!from.value || !to.value) {
    error.value = 'Debe seleccionar ambas monedas a cambiar'
    return
  }
  if (from.value === to.value) {
    result.value = amount.value
    return
  }
  loading.value = true
  try {
    const { data } = await axios.get(`https://api.frankfurter.app/latest`, {
      params: {
        amount: amount.value,
        from: from.value,
        to: to.value,
      },
    })
    result.value = data.rates[to.value]
  } catch {
    error.value = 'Error al convertir. Intente nuevamente.'
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.full-width {
  width: 100%;
}
.gradient-btn {
  background: linear-gradient(90deg, #3b82f6 0%, #06b6d4 100%);
  color: white;
}
.swap-btn {
  box-shadow: 0 2px 8px 0 rgba(0, 0, 0, 0.08);
}
.result-box {
  background: #f5faff;
  border-radius: 8px;
  padding: 16px;
  margin-top: 8px;
  font-size: 1.1rem;
}
</style>
