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
  let col18 = row[18] !== undefined ? String(row[18]).replace(/'/g, "''") : ""
  let col21 = row[21] !== undefined ? String(row[21]).replace(/'/g, "''") : ""

  return `
INSERT INTO \`wp_woocommerce_order_itemmeta\` (\`order_item_id\`, \`meta_key\`, \`meta_value\`)
VALUES
('${col18}', '_product_id', '1276'),
('${col18}', '_variation_id', '1312'),
('${col18}', '_qty', '1'),
('${col18}', '_tax_class', NULL),
('${col18}', '_line_subtotal', '34'),
('${col18}', '_line_subtotal_tax', '0'),
('${col18}', '_line_total', '34'),
('${col18}', '_line_tax', '0'),
('${col18}', '_line_tax_data', 'a:2:{s:5:"total";a:0:{}s:8:"subtotal";a:0:{}}'),
('${col18}', 'pa_available-size', 'm'),
('${col18}', 'pa_style', 't-shirt'),
('${col18}', '_ywpar_total_points', '3'),
('${col18}', '_vi_wot_order_item_tracking_data', '[{"tracking_number":"${col21}","carrier_slug":"royal-mail","carrier_url":"https://www.royalmail.com/track-your-item#/tracking-results/{tracking_number}","carrier_name":"Royal Mail","carrier_type":"define-carrier","time":1757090168}]');`.trim()
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
