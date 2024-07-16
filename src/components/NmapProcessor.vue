<template>
  <q-input outlined bottom-slots v-model.trim="ports" :error="errDetected" label="Порты" placeholder="554" dense>
    <template v-slot:before>
      <q-toggle v-model="showParsed" class="text-subtitle2" label="Фильтр" dense left-label />
    </template>

    <template v-slot:append>
      <div class="q-mb-xs q-gutter-sm">
        <q-radio dense v-model="filterMode" val="and" label="И" class="text-subtitle2" />
        <q-radio dense v-model="filterMode" val="or" label="ИЛИ" class="text-subtitle2" />
      </div>
      <q-icon v-if="ports !== ''" name="clear" @click="ports = ''" class="cursor-pointer" />
    </template>

    <template v-slot:after>
      <q-btn icon='refresh' @click="refreshAction" />
    </template>
  </q-input>

  <q-separator />

  <q-input v-if="!showParsed" :readonly="RawNmapOutput != ''" type="textarea" v-model.trim="RawNmapOutput" filled
    color="sred-12" label="Сырой Nmap" placeholder="Вставьте вывод Nmap сюда">

    <template v-slot:append>
      <q-icon v-if="RawNmapOutput" name="delete" @click.stop.prevent="RawNmapOutput = ''" class="cursor-pointer" />
    </template>
  </q-input>

  <q-input v-if="showParsed" class="full-height	" v-model="nmapParsedCopy" filled type="textarea" color="shred-12"
    label="Обработанный Nmap" :placeholder="nmapParsed">
    <template v-slot:append>
      <q-icon v-if="nmapParsed !== ''" name="delete" @click="nmapParsed = ''; RawNmapOutput = ''; showParsed = false"
        class="cursor-pointer" />
    </template>
  </q-input>
</template>

<script setup>
import { ref, watch } from 'vue'

const RawNmapOutput = ref('')
const nmapParsed = ref('') // "Блокнот" пользователя
const nmapParsedCopy = ref('') // Для быстрого восстановления
const showParsed = ref(false)
const ports = ref('')
const filterMode = ref('and')
const errDetected = ref(false)

// Если вставили новый вывод или порты
const parser = watch([RawNmapOutput, filterMode, ports], () => {
  parseNmap()
})

// Обновление демонстрации, если новый результат
const updater = watch(nmapParsed, () => {
  nmapParsedCopy.value = nmapParsed.value
})

const refreshAction = () => {
  if (RawNmapOutput.value == '') {
    return
  }
  nmapParsedCopy.value = nmapParsed.value
  showParsed.value = true
}

const parseNmap = () => {
  errDetected.value = false

  // Продолжаем если верные порты или они не указаны
  if (!/^$|^\d{1,5}( \d{1,5}| )*$/.test(ports.value)) {
    errDetected.value = true
    return
  }

  if (RawNmapOutput.value.length == 0) {
    return
  }

  let currentIP = null
  let portsFound = ''
  let foundAll = true
  let checkForPorts = ports.value.length == 0 ? '554' : ports.value

  nmapParsed.value = ''

  RawNmapOutput.value.split('\n').forEach(line => {
    // Вывод строки если были данные, сброс памяти и/или запоминание новго IP
    if (line.includes('Nmap scan report for ')) {
      if (portsFound != '') {
        if (filterMode.value == 'and') {
          checkForPorts.split(' ').forEach(port => {
            if (!portsFound.includes(port)) {
              foundAll = false
            }
          })
        }

        if (foundAll) {
          nmapParsed.value += currentIP += ': ' + portsFound.substring(0, portsFound.length - 2) + '\n'
        }

        foundAll = true
        portsFound = ''
      }
      currentIP = line.substring(21)
      return
    }

    // Запоминание открытых портов
    if (currentIP && line.includes('open')) {
      checkForPorts.split(' ').forEach(port => {
        // Доп. символы чтобы не было путаницы с 80/http и 8080/http и т.п.
        if ((' ' + line).includes(' ' + port.toString() + '/')) {
          portsFound += port + ', '
          return
        }
      })
    }
  });

  if (nmapParsed.value.length == 0) {
    nmapParsed.value = 'Совпадений не найдено'
  }

  showParsed.value = true
}
</script>
