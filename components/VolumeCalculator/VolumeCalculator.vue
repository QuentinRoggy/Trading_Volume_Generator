<!-- components/VolumeCalculator.vue -->
<template>
  <UCard>
    <template #header>
      <h3 class="text-lg font-semibold">Calculateur de volume pour cTrader</h3>
    </template>

    <UForm :state="form" @submit="calculateVolume">
      <UFormGroup label="Taille du compte ($)" name="accountSize">
        <UInput v-model="form.accountSize" type="number" placeholder="25500" />
      </UFormGroup>

      <UFormGroup label="Risque maximum (%)" name="riskPercentage">
        <UInput v-model="form.riskPercentage" type="number" placeholder="1" />
      </UFormGroup>

      <UFormGroup label="Stop Loss (en pips/points)" name="stopLoss">
        <UInput v-model="form.stopLoss" type="number" />
      </UFormGroup>

      <UFormGroup label="Instrument" name="instrument">
        <USelect v-model="form.instrument" :options="instruments" />
      </UFormGroup>

      <UButton type="submit" color="primary" class="mt-4">Calculer le volume</UButton>
    </UForm>

    <UAlert v-if="showResult" :title="'Résultat du calcul'" class="mt-4">
      <template #description>
        Volume pour cTrader: {{ cTraderVolume }}
        <br>
        Lots standards: {{ standardLots }}
      </template>
    </UAlert>
  </UCard>
</template>

<script setup>
import { ref, reactive } from 'vue'

const form = reactive({
  accountSize: 25500,
  riskPercentage: 1,
  stopLoss: 0,
  instrument: 'forex'
})

const instruments = [
  { label: 'Forex', value: 'forex' },
  { label: 'Or (XAUUSD)', value: 'gold' }
]

const showResult = ref(false)
const cTraderVolume = ref(0)
const standardLots = ref(0)

const calculateVolume = () => {
  const riskAmount = form.accountSize * (form.riskPercentage / 100)
  let volume

  if (form.instrument === 'forex') {
    // Pour le Forex, 1 pip = 10$ pour un lot standard de 100,000 unités
    volume = riskAmount / (form.stopLoss * 10)
    standardLots.value = volume.toFixed(2)
    cTraderVolume.value = Math.floor(volume * 100000)
  } else if (form.instrument === 'gold') {
    // Pour l'or, 1 point = 1$ pour 1 once, mais cTrader utilise une échelle de 10
    volume = riskAmount / form.stopLoss
    standardLots.value = (volume / 100).toFixed(2) // 1 lot standard = 100 onces
    cTraderVolume.value = Math.floor(volume * 10) // Multiplié par 10 pour l'échelle de cTrader
  }

  showResult.value = true
}
</script>