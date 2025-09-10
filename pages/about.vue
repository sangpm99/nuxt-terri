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
  const safe = (val) =>
    val !== undefined ? String(val).replace(/'/g, "''") : "";

  return `
INSERT INTO \`wp_wc_order_stats\` (
    \`order_id\`,
    \`parent_id\`,
    \`date_created\`,
    \`date_created_gmt\`,
    \`date_paid\`,
    \`date_completed\`,
    \`num_items_sold\`,
    \`total_sales\`,
    \`tax_total\`,
    \`shipping_total\`,
    \`net_total\`,
    \`returning_customer\`,
    \`status\`,
    \`customer_id\`
)
VALUES (
    '${safe(row[0])}',
    '0',
    '${safe(row[3])}',
    '${safe(row[3])}',
    '${safe(row[3])}',
    '${safe(row[3])}',
    '${safe(row[7])}',
    '${safe(row[8])}',
    '0',
    '${safe(row[9])}',
    '${safe(row[10])}',
    '1',
    'wc_completed',
    '1'
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
