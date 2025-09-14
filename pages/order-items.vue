<template>
  <div class="p-4 space-y-3">
    <input type="file" accept=".csv" @change="onFileChange" />
    <button
      @click="downloadSQL"
      :disabled="parsedData.length === 0"
      class="ml-2 px-3 py-1 border rounded"
    >
      Download SQL CSV
    </button>
  </div>
</template>

<script setup>
import { ref } from "vue"
import Papa from "papaparse"

const file = ref(null)
const parsedData = ref([])

function onFileChange(e) {
  file.value = e.target.files[0] || null
  parsedData.value = []
  if (file.value) {
    Papa.parse(file.value, {
      header: false,
      skipEmptyLines: true,
      complete: (results) => {
        parsedData.value = results.data
      },
      error: (err) => {
        console.error("Lỗi parse:", err)
      },
    })
  }
}

function buildSQL(row) {
  // lấy các cột cần dùng
  let col0 = row[0] !== undefined ? String(row[0]).replace(/'/g, "''") : ""
  let col18 = row[18] !== undefined ? String(row[18]).replace(/'/g, "''") : ""
  let col19 = row[19] !== undefined ? String(row[19]).replace(/'/g, "''") : ""
  let col20 = row[20] !== undefined ? String(row[20]).replace(/'/g, "''") : ""

  return `
INSERT INTO \`wp_woocommerce_order_items\` (\`order_item_id\`, \`order_item_name\`, \`order_item_type\`, \`order_id\`)
VALUES
('${col18}', 'Lively Design T-shirt Cat Lover - The Spoiled Rotten Cats - Personalized Unisex T-Shirt', 'line_item', '${col0}'),
('${col19}', 'Paypal / CC - Fee', 'fee', '${col0}'),
('${col20}', 'UK Warehouse 1', 'shipping', '${col0}');`.trim()
}

function downloadSQL() {
  if (parsedData.value.length === 0) return
  const sqlLines = parsedData.value.map((row) => [buildSQL(row)])
  const csvContent = Papa.unparse(sqlLines, { quotes: true })
  const blob = new Blob([csvContent], { type: "text/csv;charset=utf-8;" })
  const url = URL.createObjectURL(blob)
  const link = document.createElement("a")
  link.setAttribute("href", url)
  link.setAttribute("download", "sql_statements.csv")
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
  URL.revokeObjectURL(url)
}
</script>
