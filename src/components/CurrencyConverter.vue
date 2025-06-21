<template>
  <div class="currency-bg flex flex-center q-py-xl">
    <q-card class="q-pa-xl q-mx-auto shadow-10 currency-card" style="max-width: 420px">
      <q-card-section class="q-pb-none flex flex-center column">
        <q-icon name="account_balance_wallet" color="primary" size="40px" class="q-mb-xs" />
        <div class="text-h5 text-weight-bold text-primary">Conversión de Monedas</div>
        <div class="text-caption text-grey-7 q-mt-xs">Actualizado en tiempo real</div>
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
            color="primary"
            class="input-light"
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
                color="primary"
                class="input-light"
              />
            </div>
            <div class="col-2 flex flex-center">
              <q-btn
                round
                color="primary"
                text-color="white"
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
                color="primary"
                class="input-light"
              />
            </div>
          </div>
          <q-btn
            type="submit"
            label="Convertir"
            class="full-width gradient-btn q-mt-md"
            :loading="loading"
            unelevated
            icon="autorenew"
          />
        </q-card-section>
      </q-form>
      <q-card-section>
        <transition name="bounce-fade">
          <div
            v-if="result !== null"
            class="result-box text-center animate__animated animate__fadeInUp animate__faster"
            :key="'result-' + result"
          >
            <q-icon
              name="paid"
              color="primary"
              size="32px"
              class="q-mb-sm animate__animated animate__tada animate__delay-1s"
            />
            <span class="text-h6 text-weight-bold text-primary">
              {{ amount }} {{ from }} equivalen a {{ result }} {{ to }}
            </span>
          </div>
        </transition>
      </q-card-section>
      <q-card-section v-if="error">
        <q-banner dense class="bg-red-2 text-red-10">{{ error }}</q-banner>
      </q-card-section>
    </q-card>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const amount = ref(1)
const from = ref('')
const to = ref('')
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
@import 'https://cdnjs.cloudflare.com/ajax/libs/animate.css/4.1.1/animate.min.css';
.currency-bg {
  min-height: 100vh;
  background: linear-gradient(135deg, #f8fafc 0%, #e0e7ef 100%);
}
.currency-card {
  background: #fff;
  border-radius: 18px;
  box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.1);
  border: 1px solid #e0e7ef;
}
.input-light .q-field__control {
  background: #f3f6fa;
  color: #222;
}
.input-light .q-field__label {
  color: #7b8a99;
}
.input-light .q-field__native {
  color: #222;
}
.gradient-btn {
  background: linear-gradient(90deg, #60a5fa 0%, #38bdf8 100%);
  color: white;
  font-weight: bold;
  letter-spacing: 1px;
}
.swap-btn {
  box-shadow: 0 2px 8px 0 rgba(59, 130, 246, 0.1);
}
.result-box {
  background: #f3f6fa;
  border-radius: 8px;
  padding: 16px;
  margin-top: 8px;
  font-size: 1.1rem;
  color: #222;
  border: 1px solid #60a5fa;
  min-height: 70px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
.bounce-fade-enter-active {
  animation: bounceIn 0.7s;
}
.bounce-fade-leave-active {
  animation: fadeOut 0.3s;
}
@keyframes bounceIn {
  0% {
    opacity: 0;
    transform: scale(0.7);
  }
  60% {
    opacity: 1;
    transform: scale(1.1);
  }
  80% {
    transform: scale(0.95);
  }
  100% {
    opacity: 1;
    transform: scale(1);
  }
}
@keyframes fadeOut {
  to {
    opacity: 0;
    transform: translateY(20px);
  }
}
</style>
