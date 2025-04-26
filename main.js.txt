// main.js

document.addEventListener('DOMContentLoaded', () => {
  const form = document.getElementById('entry-form');
  
  form.addEventListener('submit', event => {
    event.preventDefault();
    addEntry();
  });

  renderAll();
});

function getEntries() {
  const raw = localStorage.getItem('weightEntries');
  return raw ? JSON.parse(raw) : [];
}

function saveEntries(entries) {
  localStorage.setItem('weightEntries', JSON.stringify(entries));
}

function addEntry() {
  const dateInput = document.getElementById('date-input').value;
  const weightInput = parseFloat(document.getElementById('weight-input').value);
  if (!dateInput || isNaN(weightInput)) return;

  const entries = getEntries();
  entries.push({ date: dateInput, weight: weightInput });
  // Sort by date ascending
  entries.sort((a, b) => a.date.localeCompare(b.date));
  saveEntries(entries);

  // Clear form
  document.getElementById('entry-form').reset();

  renderAll();
}

function renderAll() {
  const entries = getEntries();
  renderTable(entries);
  renderChart(entries);
}

function renderTable(entries) {
  const tbody = document.querySelector('#entries-table tbody');
  tbody.innerHTML = '';

  entries.forEach(entry => {
    const tr = document.createElement('tr');
    const tdDate = document.createElement('td');
    tdDate.textContent = entry.date;
    const tdWeight = document.createElement('td');
    tdWeight.textContent = entry.weight;

    tr.appendChild(tdDate);
    tr.appendChild(tdWeight);
    tbody.appendChild(tr);
  });
}

function renderChart(entries) {
  const ctx = document.getElementById('weightChart').getContext('2d');
  const labels = entries.map(e => e.date);
  const data = entries.map(e => e.weight);

  if (window.weightChart) {
    window.weightChart.data.labels = labels;
    window.weightChart.data.datasets[0].data = data;
    window.weightChart.update();
  } else {
    window.weightChart = new Chart(ctx, {
      type: 'line',
      data: {
        labels: labels,
        datasets: [{
          label: 'Weight (kg)',
          data: data,
          fill: false,
          tension: 0.1
        }]
      },
      options: {
        responsive: true,
        scales: {
          x: {
            title: { display: true, text: 'Date' }
          },
          y: {
            title: { display: true, text: 'Weight (kg)' }
          }
        }
      }
    });
  }
}
