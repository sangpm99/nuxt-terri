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
  // lấy cột 0 và cột 17, nếu thiếu thì để rỗng
  let col0 = row[0] !== undefined ? String(row[0]).replace(/'/g, "''") : ""
  let col17 = row[17] !== undefined ? String(row[17]).replace(/'/g, "''") : ""

  return `
INSERT INTO \`wp_wc_orders_meta\` (\`order_id\`, \`meta_key\`, \`meta_value\`)
VALUES
('${col0}', '_alg_wc_custom_order_number', '1899227276'),
('${col0}', '_alg_wc_full_custom_order_number', '${col17}'),
('${col0}', '_billing_address_index', 'Michael Blundell  6 Newby Close  Southport  PR8 3TW GB mgblundell@yahoo.com +447711586396'),
('${col0}', '_cs_paypal_checkout_page', 'checkout'),
('${col0}', '_cs_paypal_currency', 'GBP'),
('${col0}', '_cs_paypal_fee', '2.27'),
('${col0}', '_cs_paypal_intent', 'CAPTURE'),
('${col0}', '_cs_paypal_payout', '42.54'),
('${col0}', '_edit_lock', '1757142372:1'),
('${col0}', '_mecom_paypal_proxy_id', '68bb091a90471'),
('${col0}', '_mecom_paypal_proxy_url', 'https://terrionahill.us'),
('${col0}', '_shield_payment_method', 'paypal'),
('${col0}', '_shield_payment_url', 'https://terrionahill.us'),
('${col0}', '_shield_paypal_funding_source', 'card'),
('${col0}', '_shipping_address_index', 'Michael Blundell  6 Newby Close  Southport Merseyside PR8 3TW GB +447711586396'),
('${col0}', '_wc_order_attribution_device_type', 'Mobile'),
('${col0}', '_wc_order_attribution_session_count', '1'),
('${col0}', '_wc_order_attribution_session_entry', 'http://terrionahill2.local/'),
('${col0}', '_wc_order_attribution_session_pages', '23'),
('${col0}', '_wc_order_attribution_session_start_time', '2025-09-05 15:31:04'),
('${col0}', '_wc_order_attribution_source_type', 'typein'),
('${col0}', '_wc_order_attribution_user_agent', 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36'),
('${col0}', '_wc_order_attribution_utm_source', '(direct)'),
('${col0}', '_ywpar_conversion_points', 'a:2:{s:6:"points";i:1;s:5:"money";i:10;}'),
('${col0}', '_ywpar_points_earned', '3'),
('${col0}', 'is_vat_exempt', 'no'),
('${col0}', 'METAKEY_CS_TRANSACTION_ID', '0F321733VF256745S'),
('${col0}', 'ywpar_points_from_cart', '3');`.trim()
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
