<script setup>
import Papa from "papaparse";
import { ref } from "vue";

const tableName = ref("my_table");
// input thêm: các cột cần giữ NULL
const nullColsInput = ref("");

function handleFile(e) {
  const file = e.target.files[0];

  // parse input string thành array tên cột
  const nullCols = nullColsInput.value
    .split(",")
    .map((s) => s.trim())
    .filter((s) => s);

  Papa.parse(file, {
    header: true,
    skipEmptyLines: true,
    complete: (results) => {
      const headers = results.meta.fields;
      const rows = results.data;

      // hàm xử lý giá trị theo cột
      const esc = (col, v) => {
        const isEmpty = v === null || v === undefined || v === "";
        if (nullCols.includes(col)) {
          return isEmpty ? "NULL" : `'${String(v).replace(/'/g, "''")}'`;
        } else {
          return v === null || v === undefined
            ? "''"
            : `'${String(v).replace(/'/g, "''")}'`;
        }
      };

      const values = rows
        .map(
          (row) =>
            `(${headers.map((h) => esc(h, row[h])).join(",")})`
        )
        .join(",\n");

      const sql = `INSERT INTO ${tableName.value} (${headers.join(
        ","
      )}) VALUES\n${values};`;

      downloadFile(sql, "output.sql");
    },
  });
}

function downloadFile(content, filename) {
  const blob = new Blob([content], { type: "text/sql" });
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = filename;
  a.click();
  URL.revokeObjectURL(url);
}
</script>

<template>
  <input v-model="tableName" placeholder="Tên table" />
  <input v-model="nullColsInput" placeholder="Các cột NULL (cách nhau bằng ,)" />
  <input type="file" accept=".csv" @change="handleFile" />
</template>
