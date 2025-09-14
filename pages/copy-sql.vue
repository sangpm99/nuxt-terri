<template>
  <div class="p-4 space-y-3">
    <input type="file" accept=".csv" @change="onFileChange" />

    <div v-if="joinedData.length > 0" class="space-y-2">
      <button
        v-for="(line, idx) in joinedData"
        :key="idx"
        @click="copyToClipboard(line)"
        class="w-full px-3 py-1 border rounded bg-gray-100 hover:bg-gray-200 text-left"
      >
        Copy dòng {{ idx + 1 }}
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue"
import Papa from "papaparse"

const file = ref(null)
const joinedData = ref([])

function onFileChange(e) {
  file.value = e.target.files[0] || null
  joinedData.value = []

  if (file.value) {
    Papa.parse(file.value, {
      header: false,
      skipEmptyLines: true,
      complete: (results) => {
        joinedData.value = results.data.map((row) => {
          // lấy cột 0,1,2,3
          let cols = []
          for (let i = 0; i <= 3; i++) {
            cols.push(row[i] !== undefined ? String(row[i]) : "")
          }
          return cols.join(" ").trim()
        }).filter(line => line.length > 0)
      },
      error: (err) => {
        console.error("Lỗi parse:", err)
      },
    })
  }
}

function copyToClipboard(text) {
  if (!text) return
  navigator.clipboard.writeText(text).then(
    () => console.log("Copied:", text),
    (err) => console.error("Copy failed:", err)
  )
}
</script>
