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
  const cols = []
  for (let i = 0; i <= 15; i++) {
    let val = row[i] !== undefined ? String(row[i]) : ""
    // escape dấu nháy đơn
    val = val.replace(/'/g, "''")
    cols.push(val)
  }

  return `
INSERT INTO \`wp_wc_orders\` (
    \`id\`,
    \`status\`,
    \`currency\`,
    \`type\`,
    \`tax_amount\`,
    \`total_amount\`,
    \`customer_id\`,
    \`billing_email\`,
    \`date_created_gmt\`,
    \`date_updated_gmt\`,
    \`parent_order_id\`,
    \`payment_method\`,
    \`payment_method_title\`,
    \`transaction_id\`,
    \`ip_address\`,
    \`user_agent\`,
    \`customer_note\`
)
VALUES (
    '${cols[0]}',
    '${cols[1]}',
    '${cols[2]}',
    '${cols[3]}',
    '${cols[4]}',
    '${cols[5]}',
    '${cols[6]}',
    '${cols[7]}',
    '${cols[8]}',
    '${cols[9]}',
    '${cols[10]}',
    '${cols[11]}',
    '${cols[12]}',
    '${cols[13]}',
    '${cols[14]}',
    '${cols[15]}',
    NULL
);`.trim()
}

function downloadSQL() {
  if (parsedData.value.length === 0) return

  // xây list SQL
  const sqlLines = parsedData.value.map((row) => [buildSQL(row)])

  // convert sang CSV với 1 cột
  const csvContent = Papa.unparse(sqlLines, {
    quotes: true,
  })

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
