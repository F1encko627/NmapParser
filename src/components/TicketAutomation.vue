<template>
  <q-input type="textarea" placeholder="Вставьте вывод srv-global сюда" filled color="sred-12"
    v-model.trim="actualTickets" label="Актуальные тикеты">

    <template v-if="actualTickets != ''" v-slot:append>
      <q-icon name="clear" @click.stop.prevent="actualTickets = ''" class="cursor-pointer" />
    </template>

  </q-input>

  <q-btn label="Обновить данные" class="full-width" @click="compareTickets" />

  <div class="text-h6 q-my-md">Новые тикеты</div>
  <q-input type="textarea" filled color="sred-12" v-model="newTickets">

    <template v-if="newTickets != ''" v-slot:append>
      <q-icon name="clear" @click.stop.prevent="newTickets = ''" class="cursor-pointer" />
    </template>

  </q-input>

  <div class="text-h6 q-my-md">Оставшиеся тикеты</div>
  <q-input type="textarea" filled color="sred-12" v-model="oldTickets">

    <template v-if="oldTickets != ''" v-slot:append>
      <q-icon name="clear" @click.stop.prevent="oldTickets = ''" class="cursor-pointer" />
    </template>

  </q-input>

  <div class="text-h6 q-my-md">Исчезнувшие тикеты</div>
  <q-input type="textarea" filled color="sred-12" v-model="srdTickets">

    <template v-if="srdTickets != ''" v-slot:append>
      <q-icon name="clear" @click.stop.prevent="srdTickets = ''" class="cursor-pointer" />
    </template>

  </q-input>
</template>

<script setup>
import { ref, watch } from 'vue'
import { useQuasar } from 'quasar'

const $q = useQuasar()

const actualTickets = ref('')
var currentTickets = new Set() // последние добавленные данные
var lastTickets = new Set() // прошлые добавленные данные
const newTickets = ref('')
const oldTickets = ref('')
const srdTickets = ref('') // selfresolved

const checkForValidIp = text => {
  return /^(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/.test(text)
}

const compareTickets = () => {
  lastTickets = new Set(currentTickets)
  currentTickets.clear()

  let elN = 1
  let tmp = ''

  actualTickets.value.split('\n').forEach(el => {
    switch (elN) {
      case 1:
        tmp = el + ' // '
        elN++
        return
      case 2:
        if (checkForValidIp(el)) {
          tmp += ' -- НЕТ ПОДРАЗДЕЛЕНИЯ -- // ' + el
          elN = 1
          break
        }
        tmp += el + ' // '
        elN++
        return
      case 3:
        tmp += el
        elN = 1
    }

    currentTickets.add(tmp)
  })

  newTickets.value = Array.from(currentTickets.difference(lastTickets)).join('\n')
  oldTickets.value = Array.from(currentTickets.intersection(lastTickets)).join('\n')
  srdTickets.value = Array.from(lastTickets.difference(currentTickets)).join('\n')
}
</script>
