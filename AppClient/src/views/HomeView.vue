<script setup lang="ts">
import { LogLevel, HubConnectionBuilder } from '@microsoft/signalr'
import { ref, onMounted } from 'vue'

// reactive state
const name = ref()
const submitted = ref(false)
const message = ref()
const messages = ref([])

const connection = new HubConnectionBuilder()
  .withUrl('http://localhost:5159/chat')
  .configureLogging(LogLevel.Information)
  .build()

const start = async () => {
  try {
    await connection.start()
    console.log('SignalR Connected.')
  } catch (err) {
    console.log(err)
    setTimeout(start, 5000)
  }
}

connection.onclose(async () => {
  await start()
})

connection.on('ReceiveMessage', (name: string, message: string) => {
  if (!submitted.value) return

  messages.value.push({
    name,
    message
  })
})

const onSubmit = () => {
  if (!submitted.value) submitted.value = true
}

const onSend = async () => {
  await connection.send('SendMessage', name.value, message.value)
  message.value = ''
}

// lifecycle hooks
onMounted(async () => {
  await start()
})
</script>

<template>
  <div class="about">
    <el-form :inline="true" v-if="!submitted">
      <el-form-item>
        <el-input v-model="name" placeholder="Name" />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="onSubmit">Submit</el-button>
      </el-form-item>
    </el-form>
    <el-form :inline="true" v-else>
      <el-form-item>
        <el-input v-model="message" placeholder="Message" />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="onSend">Send</el-button>
      </el-form-item>
      <el-divider />
      <el-row v-for="(m, index) in messages" :key="index">
        <el-col :span="24">
          <el-text>
            <el-text tag="b">{{ m.name }}</el-text
            >: {{ m.message }}
          </el-text>
        </el-col>
      </el-row>
    </el-form>
  </div>
</template>

<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>
