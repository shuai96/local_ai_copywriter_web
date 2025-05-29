<template>
  <div class="min-h-screen bg-gray-100 p-6">
    <div class="max-w-2xl mx-auto bg-white rounded-2xl shadow-lg p-8 space-y-6">
      <h1 class="text-2xl font-bold text-gray-800">AI文案生成器</h1>

      <div class="space-y-4">
        <div>
          <label class="block text-sm font-medium text-gray-700">产品名称</label>
          <input
            v-model="form.product_name"
            class="w-full mt-1 p-2 border rounded-xl"
            placeholder="如 蓝牙耳机"
          />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-700">产品卖点</label>
          <textarea
            v-model="form.product_features"
            class="w-full mt-1 p-2 border rounded-xl"
            rows="3"
            placeholder="如 高清音质，佩戴舒适，续航10小时"
          ></textarea>
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-700">风格与字数</label>
          <input
            v-model="form.style"
            class="w-full mt-1 p-2 border rounded-xl"
            placeholder="如 小红书种草风，500字"
          />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-700">模型选择</label>
          <select v-model="form.model" class="w-full mt-1 p-2 border rounded-xl">
            <option :value="1">deepseek-r1</option>
            <option :value="2">mistral</option>
          </select>
        </div>

        <button
          @click="submit"
          class="w-full py-2 px-4 bg-blue-600 text-white font-semibold rounded-xl hover:bg-blue-700"
          :disabled="loading"
        >
          {{ loading ? '生成中...' : '生成文案' }}
        </button>
      </div>

      <div v-if="output" class="mt-6">
        <h2 class="text-xl font-semibold text-gray-800 mb-2">生成的文案：</h2>
        <div class="whitespace-pre-wrap bg-gray-50 border p-4 rounded-xl text-gray-700">
          {{ output }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const form = ref({
  product_name: '',
  product_features: '',
  style: '',
  model: 1,
})

const output = ref('')
const loading = ref(false)

const submit = async () => {
  loading.value = true
  output.value = ''
  try {
    const response = await fetch('http://127.0.0.1:8000/ai/stream-generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(form.value),
    })
    if (!response.body) throw new Error('无响应体')
    const reader = response.body.getReader()
    const decoder = new TextDecoder('utf-8')
    let done = false
    while (!done) {
      const { value, done: doneReading } = await reader.read()
      done = doneReading
      if (value) {
        output.value += decoder.decode(value)
      }
    }
  } catch (error) {
    output.value = '请求失败，请检查后端或网络连接'
    console.error(error)
  } finally {
    loading.value = false
  }
}
</script>
