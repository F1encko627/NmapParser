<template>
  <q-tabs v-model="tab" dense :class="$q.dark.isActive ? 'bg-grey-9' : 'bg-grey-3'" align="justify" narrow-indicator>
    <q-tab name="1" label="Обработчик Nmap" />
    <q-tab name="2" label="Отслеживание Тикетов" />
    <q-btn icon="light_mode" />
    <q-btn icon="dark_mode" />
  </q-tabs>

  <q-separator />

  <q-tab-panels v-model="tab" animated>
    // Панель фильтрации
    <q-tab-panel name="1">
      <NmapProcessor />
    </q-tab-panel>

    // Панель отслеживания
    <q-tab-panel name="2">
      <TicketAutomation />
    </q-tab-panel>

  </q-tab-panels>

</template>

<script setup>
import { ref, watch } from 'vue'
import { useQuasar } from 'quasar'
import TicketAutomation from '../components/TicketAutomation.vue'
import NmapProcessor from '../components/NmapProcessor.vue'

const $q = useQuasar()

const tab = ref('1')
</script>
