#!/usr/bin/env python3
"""
SQM Costing Calculator - HTML Generator
Generates a standalone, self-contained HTML file for SQM print costing.
Features: blank start, material column, live calculations, PDF/CSV export.
"""

HTML_TEMPLATE = r"""<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>SQM Costing Calculator</title>
<style>
  @media print {
    body { background: #fff; padding: 0; }
    .no-print { display: none !important; }
    .container { max-width: 100%; }
    table { box-shadow: none; border: 1px solid #ccc; }
    .summary { box-shadow: none; border: 1px solid #ccc; margin-left: 0; max-width: 100%; }
  }
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
    background: #f5f5f7;
    color: #1d1d1f;
    padding: 24px;
    line-height: 1.5;
  }
  .container { max-width: 1100px; margin: 0 auto; }
  h1 { font-size: 24px; font-weight: 600; margin-bottom: 20px; }
  .toolbar {
    display: flex; justify-content: space-between; align-items: center;
    margin-bottom: 16px; flex-wrap: wrap; gap: 12px;
  }
  .toolbar-group { display: flex; align-items: center; gap: 8px; }
  label { font-size: 14px; color: #6e6e73; }
  input[type="number"], input[type="text"] {
    padding: 8px 10px; border: 1px solid #d2d2d7; border-radius: 8px;
    font-size: 14px; background: #fff; color: #1d1d1f;
  }
  input[type="number"] { text-align: center; width: 70px; }
  table { width: 100%; border-collapse: collapse; background: #fff; border-radius: 12px; overflow: hidden; box-shadow: 0 1px 3px rgba(0,0,0,0.06); }
  th, td { padding: 10px 12px; font-size: 14px; }
  th {
    background: #fafafa; color: #6e6e73; font-weight: 500;
    text-align: center; border-bottom: 1px solid #e5e5ea;
    white-space: nowrap;
  }
  th:first-child, th:nth-child(2) { text-align: left; }
  td { border-bottom: 1px solid #f0f0f0; }
  td input { width: 100%; border: 1px solid transparent; background: transparent; padding: 6px; border-radius: 6px; font-size: 14px; }
  td input:focus { outline: none; border-color: #0071e3; background: #fff; }
  td input[data-field="name"] { text-align: left; min-width: 180px; }
  td input[data-field="material"] { text-align: left; min-width: 120px; }
  td input[data-field="w"], td input[data-field="h"], td input[data-field="qty"] { text-align: center; width: 70px; }
  td input[data-field="price"] { text-align: right; width: 80px; }
  .num { text-align: right; font-variant-numeric: tabular-nums; }
  .line-total { font-weight: 600; }
  tr:hover { background: #f9f9fb; }
  .row-actions { opacity: 0; transition: opacity 0.15s ease; }
  tr:hover .row-actions { opacity: 1; }
  .btn-add {
    margin-top: 12px; padding: 8px 16px; border: 1px dashed #d2d2d7;
    border-radius: 8px; background: transparent; color: #6e6e73;
    font-size: 14px; cursor: pointer; display: inline-flex; align-items: center; gap: 6px;
  }
  .btn-add:hover { border-color: #1d1d1f; color: #1d1d1f; }
  .summary {
    margin-top: 24px; padding: 20px; border-radius: 12px;
    background: #fff; box-shadow: 0 1px 3px rgba(0,0,0,0.06);
    max-width: 360px; margin-left: auto;
  }
  .summary-row { display: flex; justify-content: space-between; font-size: 14px; margin-bottom: 8px; }
  .summary-row span:first-child { color: #6e6e73; }
  .summary-row span:last-child { font-weight: 500; }
  .summary-total {
    border-top: 1px solid #e5e5ea; padding-top: 12px; margin-top: 8px;
    display: flex; justify-content: space-between; font-size: 22px; font-weight: 600;
  }
  .btn-del {
    background: none; border: none; cursor: pointer; color: #ff3b30;
    padding: 4px; border-radius: 6px;
  }
  .btn-del:hover { background: rgba(255,59,48,0.08); }
  .btn-export, .btn-clear, .btn-pdf {
    padding: 8px 16px; border: none; border-radius: 8px;
    font-size: 14px; cursor: pointer;
  }
  .btn-export { background: #0071e3; color: #fff; }
  .btn-export:hover { background: #005bb5; }
  .btn-clear { background: #ff3b30; color: #fff; margin-right: 8px; }
  .btn-clear:hover { background: #d32f2f; }
  .btn-pdf { background: #34c759; color: #fff; margin-right: 8px; }
  .btn-pdf:hover { background: #248a3d; }
  .empty-state {
    text-align: center; padding: 40px; color: #6e6e73; font-size: 14px;
  }
  @media (max-width: 768px) {
    .toolbar { flex-direction: column; align-items: flex-start; }
    table { font-size: 13px; }
    th, td { padding: 8px 6px; }
  }
</style>
</head>
<body>
<div class="container">
  <h1>SQM Costing Calculator</h1>

  <div class="toolbar no-print">
    <div class="toolbar-group">
      <label>VAT %</label>
      <input id="vatRate" type="number" value="20" min="0" max="100" onchange="recalcAll()">
      <label style="margin-left:12px">Unit price</label>
      <input id="globalPrice" type="number" value="285" min="0" onchange="applyGlobalPrice()">
    </div>
    <div>
      <button class="btn-clear" onclick="clearAll()">Clear all</button>
      <button class="btn-pdf" onclick="exportPDF()">Export PDF</button>
      <button class="btn-export" onclick="exportCSV()">Export CSV</button>
    </div>
  </div>

  <div style="overflow-x:auto;">
    <table>
      <thead>
        <tr>
          <th>Item name</th>
          <th>Material</th>
          <th>Width (mm)</th>
          <th>Height (mm)</th>
          <th>Qty</th>
          <th>SQM</th>
          <th>Total SQM</th>
          <th>Price / SQM</th>
          <th>Line total</th>
          <th class="no-print"></th>
        </tr>
      </thead>
      <tbody id="tableBody"></tbody>
    </table>
  </div>

  <button class="btn-add no-print" onclick="addRow()">
    <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/></svg>
    Add item
  </button>

  <div class="summary">
    <div class="summary-row">
      <span>Subtotal</span>
      <span id="subtotal">0.00</span>
    </div>
    <div class="summary-row">
      <span>VAT (<span id="vatLabel">20</span>%)</span>
      <span id="vatAmount">0.00</span>
    </div>
    <div class="summary-total">
      <span>Total</span>
      <span id="grandTotal">0.00</span>
    </div>
  </div>
</div>

<script>
let rows = [];

function fmt(n) {
  return Number(n).toLocaleString("en-US", { minimumFractionDigits: 2, maximumFractionDigits: 2 });
}
function fmt4(n) {
  return Number(n).toLocaleString("en-US", { minimumFractionDigits: 4, maximumFractionDigits: 4 });
}

function createRow(data, index) {
  const sqm = (data.w * data.h) / 1000000;
  const totalSqm = sqm * data.qty;
  const price = data.price || 285;
  const lineTotal = totalSqm * price;

  const tr = document.createElement("tr");
  tr.innerHTML = `
    <td><input type="text" value="${data.name}" data-field="name" onchange="updateRow(${index})"></td>
    <td><input type="text" value="${data.material || ""}" data-field="material" placeholder="e.g. vinyl" onchange="updateRow(${index})"></td>
    <td><input type="number" value="${data.w}" data-field="w" onchange="updateRow(${index})"></td>
    <td><input type="number" value="${data.h}" data-field="h" onchange="updateRow(${index})"></td>
    <td><input type="number" value="${data.qty}" data-field="qty" onchange="updateRow(${index})"></td>
    <td class="num" data-out="sqm">${fmt4(sqm)}</td>
    <td class="num" data-out="totalSqm">${fmt4(totalSqm)}</td>
    <td><input type="number" value="${price}" data-field="price" onchange="updateRow(${index})"></td>
    <td class="num line-total" data-out="lineTotal">${fmt(lineTotal)}</td>
    <td class="row-actions no-print"><button class="btn-del" onclick="removeRow(${index})"><svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="3 6 5 6 21 6"/><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"/></svg></button></td>
  `;
  return tr;
}

function render() {
  const tbody = document.getElementById("tableBody");
  tbody.innerHTML = "";
  if (rows.length === 0) {
    const tr = document.createElement("tr");
    tr.innerHTML = `<td colspan="10" class="empty-state">No items yet. Click "Add item" to start.</td>`;
    tbody.appendChild(tr);
  } else {
    rows.forEach((row, i) => tbody.appendChild(createRow(row, i)));
  }
  recalcTotals();
}

function updateRow(index) {
  const tbody = document.getElementById("tableBody");
  const tr = tbody.children[index];
  const name = tr.querySelector('[data-field="name"]').value;
  const material = tr.querySelector('[data-field="material"]').value;
  const w = parseFloat(tr.querySelector('[data-field="w"]').value) || 0;
  const h = parseFloat(tr.querySelector('[data-field="h"]').value) || 0;
  const qty = parseFloat(tr.querySelector('[data-field="qty"]').value) || 0;
  const price = parseFloat(tr.querySelector('[data-field="price"]').value) || 0;
  rows[index] = { name, material, w, h, qty, price };

  const sqm = (w * h) / 1000000;
  const totalSqm = sqm * qty;
  const lineTotal = totalSqm * price;

  tr.querySelector('[data-out="sqm"]').textContent = fmt4(sqm);
  tr.querySelector('[data-out="totalSqm"]').textContent = fmt4(totalSqm);
  tr.querySelector('[data-out="lineTotal"]').textContent = fmt(lineTotal);
  recalcTotals();
}

function recalcTotals() {
  let subtotal = 0;
  rows.forEach(r => {
    const sqm = (r.w * r.h) / 1000000;
    subtotal += sqm * r.qty * (r.price || 285);
  });
  const vatRate = parseFloat(document.getElementById("vatRate").value) || 0;
  const vat = subtotal * (vatRate / 100);
  document.getElementById("subtotal").textContent = fmt(subtotal);
  document.getElementById("vatLabel").textContent = vatRate;
  document.getElementById("vatAmount").textContent = fmt(vat);
  document.getElementById("grandTotal").textContent = fmt(subtotal + vat);
}

function recalcAll() {
  if (rows.length === 0) return;
  Array.from(document.getElementById("tableBody").children).forEach((_, i) => updateRow(i));
}

function applyGlobalPrice() {
  const price = parseFloat(document.getElementById("globalPrice").value) || 0;
  rows.forEach((r, i) => rows[i].price = price);
  render();
}

function addRow() {
  rows.push({ name: "New item", material: "", w: 1000, h: 1000, qty: 1, price: parseFloat(document.getElementById("globalPrice").value) || 285 });
  render();
}

function removeRow(index) {
  rows.splice(index, 1);
  render();
}

function clearAll() {
  if (confirm("Are you sure you want to clear all items and start a new sheet?")) {
    rows = [];
    document.getElementById("vatRate").value = 20;
    document.getElementById("globalPrice").value = 285;
    render();
  }
}

function exportPDF() {
  window.print();
}

function exportCSV() {
  const vatRate = parseFloat(document.getElementById("vatRate").value) || 0;
  let csv = "Item Name,Material,Width (mm),Height (mm),Qty,SQM,Total SQM,Price/SQM,Line Total\n";
  rows.forEach(r => {
    const sqm = (r.w * r.h) / 1000000;
    const totalSqm = sqm * r.qty;
    const lineTotal = totalSqm * (r.price || 285);
    csv += `"${r.name}","${r.material || ""}",${r.w},${r.h},${r.qty},${sqm.toFixed(4)},${totalSqm.toFixed(4)},${r.price || 285},${lineTotal.toFixed(2)}\n`;
  });
  let subtotal = 0;
  rows.forEach(r => subtotal += ((r.w * r.h) / 1000000) * r.qty * (r.price || 285));
  const vat = subtotal * (vatRate / 100);
  csv += `\n,,,,,,,Subtotal,${subtotal.toFixed(2)}\n`;
  csv += `,,,,,,,VAT (${vatRate}%),${vat.toFixed(2)}\n`;
  csv += `,,,,,,,Total,${(subtotal + vat).toFixed(2)}\n`;

  const blob = new Blob([csv], { type: "text/csv" });
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = "sqm_costing.csv";
  a.click();
  URL.revokeObjectURL(url);
}

render();
</script>
</body>
</html>"""


def generate(output_path="sqm_costing_calculator.html"):
    """Write the standalone HTML calculator to the given path."""
    with open(output_path, "w", encoding="utf-8") as f:
        f.write(HTML_TEMPLATE)
    print(f"Generated: {output_path}")


# Run it
if __name__ == "__main__":
    generate()
