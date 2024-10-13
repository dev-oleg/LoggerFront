<template>
  <q-page class="page q-pa-md">
    <div class="row q-mb-lg">
      <q-input
        v-model:model-value="filePath"
        :rules="[ val => !!val || 'Это поле является обязательным' ]"
        outlined
        stack-label
        clearable
        style="width: 300px;"
        label="Укажите путь к файлу"
      />

      <q-input
        :model-value="amount"
        outlined
        stack-label
        style="width: 300px;"
        label="Укажите количество строк"
        class="q-ml-md"
        @update:model-value="updateAmount"
      >
        <template #prepend>
          <q-btn
            round
            flat
            icon="remove_circle"
            @click="subAmount"
          />
        </template>

        <template #append>
          <q-btn
            round
            flat
            icon="add_circle"
            @click="addAmount"
          />
        </template>
      </q-input>

      <q-btn
        :disable="!filePath"
        style="height: 56px;"
        color="primary"
        class="q-ml-md"
        label="Применить"
        @click="apply"
      />
    </div>

    <div>
      <q-input
        :model-value="logsList"
        outlined
        readonly
        type="textarea"
        class="output"
      />
    </div>
  </q-page>
</template>

<script setup lang="ts">
import type { QInputProps } from 'quasar'
import { ref, computed, onMounted } from 'vue'
import { useQuasar } from 'quasar'
import { api } from 'boot/axios'

const q = useQuasar()

const filePath = ref( 'e:/test/test.log' )
const amount = ref( 4 )
const logs = ref<Array<string>>([])

const logsList = computed<string>( () => logs.value.join( '\n') )

// положительные числа целые
const REGEX_NUM_POS_INT = /^\d+$/;

function updateAmount( value: QInputProps['modelValue'] ) {
  if ( !value ) {
    amount.value = 1
    return
  }

  if ( REGEX_NUM_POS_INT.test( value as string ) ) {
    amount.value = value === '0' ? 1 : +value
  }
}

function subAmount() {
  if ( amount.value === 1 ) return
  
  amount.value--
}

function addAmount() {
  amount.value++
}

async function apply() {
  logs.value = []

  try {
    await api.post( 'http://localhost:5000/tail', {
      // id: new Date().getTime(),
      filePath: filePath.value,
      nLines: amount.value
    })
  } catch ( e ) {
    q.notify({
      message: 'Ошибка',
      caption: 'Неверно указан путь к файлу',
      type: 'negative'
    })
  }
}

async function subscribe() {
  const eventSource = new EventSource( 'http://localhost:5000/connect' )

  eventSource.onmessage = function( event ) {
    const data = JSON.parse( event.data )

    logs.value.push( data )
  }

  // eventSource.onerror = function( event ) {
  //   console.log( 'error:', event )
  // }
}

onMounted( subscribe )
</script>

<style scoped lang="scss">
.page {
  max-height: calc(100vh - 50px);
  display: grid;
  grid-template-rows: max-content 1fr;
}

.output {
  height: 100%;

  :deep(.q-field__control) {
    height: 100%;

    &:before {
      border-style: solid !important;
    }

    .q-field__native {
      resize: none;
    }
  }
}
</style>
