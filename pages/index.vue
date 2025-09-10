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
import { ref } from "vue";
import Papa from "papaparse";

const file = ref(null);
const parsedData = ref([]);

function onFileChange(e) {
  file.value = e.target.files[0] || null;
  parsedData.value = [];
  if (file.value) {
    Papa.parse(file.value, {
      header: false,
      skipEmptyLines: true,
      complete: (results) => {
        parsedData.value = results.data;
      },
      error: (err) => {
        console.error("Lỗi parse:", err);
      },
    });
  }
}

function buildSQL(row) {
  // escape để tránh lỗi khi có dấu nháy đơn
  const safe = (val) =>
    val !== undefined ? String(val).replace(/'/g, "''") : "";

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
    '${safe(row[0])}',
    'wc-completed',
    'GBP',
    'shop_order',
    '0.00000000',
    '${safe(row[1])}',
    '1',
    '${safe(row[2])}',
    '${safe(row[3])}',
    '${safe(row[3])}',
    '0',
    'mecom_paypal',
    'PayPal - Credit or Debit Card',
    '${safe(row[4])}',
    '${safe(row[5])}',
    '${safe(row[6])}',
    NULL
);`.trim();
}

function downloadSQL() {
  if (parsedData.value.length === 0) return;

  // nối toàn bộ câu SQL thành 1 file .sql
  const sqlContent = parsedData.value.map((row) => buildSQL(row)).join("\n\n");

  const blob = new Blob([sqlContent], { type: "text/sql;charset=utf-8;" });
  const url = URL.createObjectURL(blob);
  const link = document.createElement("a");
  link.setAttribute("href", url);
  link.setAttribute("download", "statements.sql");
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
  URL.revokeObjectURL(url);
}
</script>
