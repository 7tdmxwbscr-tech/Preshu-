# <!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Control de Gastos Personal</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    :root {
      --bg-color: #F4F6F9;
      --card-bg: #FFFFFF;
      --primary: #6338FF;
      --primary-light: #EBE5FF;
      --danger: #FF4D4D;
      --success: #00C853;
      --text-dark: #1A1A2E;
      --text-muted: #8E8EA1;
      --border-color: #E2E8F0;
    }

    * { box-sizing: border-box; margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; }
    body { background-color: var(--bg-color); color: var(--text-dark); padding-bottom: 80px; }

    /* HEADER */
    .header {
      background: var(--primary); color: white; padding: 20px;
      border-bottom-left-radius: 20px; border-bottom-right-radius: 20px;
      text-align: center; box-shadow: 0 4px 12px rgba(99, 56, 255, 0.2);
    }
    .header h1 { font-size: 1.3rem; font-weight: 700; }

    /* CONTENEDOR DE PÁGINAS */
    .container { padding: 16px; max-width: 480px; margin: 0 auto; }
    .page { display: none; }
    .page.active { display: block; }

    /* TARJETAS RESUMEN */
    .summary-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-bottom: 20px; }
    .summary-card {
      background: var(--card-bg); padding: 14px; border-radius: 14px;
      border: 1px solid var(--border-color); text-align: center;
    }
    .summary-card span { font-size: 0.75rem; color: var(--text-muted); text-transform: uppercase; font-weight: 700; }
    .summary-card div { font-size: 1.1rem; font-weight: 800; margin-top: 4px; }
    .text-success { color: var(--success); }
    .text-danger { color: var(--danger); }

    /* CONTENEDOR DEL GRÁFICO DE TORTA */
    .chart-box {
      background: var(--card-bg); padding: 20px; border-radius: 16px;
      border: 1px solid var(--border-color); margin-bottom: 20px;
      display: flex; flex-direction: column; align-items: center; position: relative;
    }
    .chart-container { width: 220px; height: 220px; position: relative; }

    /* FORMULARIO DE REGISTRO */
    .form-card {
      background: var(--card-bg); padding: 18px; border-radius: 16px;
      border: 1px solid var(--border-color); margin-bottom: 20px;
    }
    .form-card h3 { font-size: 0.95rem; margin-bottom: 12px; color: var(--text-dark); }
    .input-group { margin-bottom: 12px; }
    .input-group label { font-size: 0.75rem; color: var(--text-muted); font-weight: 600; display: block; margin-bottom: 4px; }
    .input-group input, .input-group select {
      width: 100%; padding: 10px 12px; border-radius: 10px; border: 1px solid var(--border-color);
      font-size: 0.9rem; outline: none; background: #FAFAFC;
    }
    .btn-submit {
      width: 100%; padding: 12px; background: var(--primary); color: white; border: none;
      border-radius: 10px; font-weight: 700; font-size: 0.95rem; cursor: pointer;
    }

    /* HISTORIAL / DESGLOSE */
    .history-list { display: flex; flex-direction: column; gap: 10px; }
    .history-item {
      background: var(--card-bg); padding: 14px; border-radius: 12px;
      border: 1px solid var(--border-color); display: flex; justify-content: space-between; align-items: center;
    }
    .item-info { display: flex; flex-direction: column; }
    .item-title { font-weight: 700; font-size: 0.9rem; }
    .item-date { font-size: 0.75rem; color: var(--text-muted); }
    .item-amount { font-weight: 800; font-size: 0.95rem; }
    .btn-delete { background: none; border: none; color: var(--danger); font-size: 1rem; cursor: pointer; margin-left: 10px; }

    /* BARRA DE NAVEGACIÓN INFERIOR (TAB BAR) */
    .bottom-nav {
      position: fixed; bottom: 0; left: 0; right: 0; background: var(--card-bg);
      border-top: 1px solid var(--border-color); display: flex; justify-content: space-around;
      padding: 10px 0; z-index: 100;
    }
    .nav-item {
      display: flex; flex-direction: column; align-items: center; gap: 4px;
      color: var(--text-muted); font-size: 0.75rem; font-weight: 700; cursor: pointer; text-decoration: none;
    }
    .nav-item.active { color: var(--primary); }
    .nav-icon { font-size: 1.2rem; }
  </style>
</head>
<body>

  <div class="header">
    <h1>💰 Mi Control Financiero</h1>
  </div>

  <div class="container">
    
    <!-- PÁGINA 1: INICIO Y GRÁFICO DE TORTA -->
    <div id="page-home" class="page active">
      <div class="summary-grid">
        <div class="summary-card">
          <span>Ingresos</span>
          <div id="total-income" class="text-success">$0</div>
        </div>
        <div class="summary-card">
          <span>Gastado</span>
          <div id="total-expense" class="text-danger">$0</div>
        </div>
      </div>

      <!-- GRÁFICO DE TORTA DE INGRESOS VS GASTOS -->
      <div class="chart-box">
        <h3 style="font-size:0.85rem; color:var(--text-muted); margin-bottom:12px;">USO DE INGRESOS (%)</h3>
        <div class="chart-container">
          <canvas id="budgetChart"></canvas>
        </div>
      </div>

      <!-- FORMULARIO DE CARGA RÁPIDA -->
      <div class="form-card">
        <h3>➕ Registrar Nuevo Movimiento</h3>
        <form id="finance-form">
          <div class="input-group">
            <label>Tipo</label>
            <select id="type" onchange="toggleCategoryInput()">
              <option value="expense">Gasto (-)</option>
              <option value="income">Ingreso (+)</option>
            </select>
          </div>
          <div class="input-group">
            <label>Monto ($)</label>
            <input type="number" id="amount" placeholder="Ej. 15000" step="any" required>
          </div>
          <div class="input-group" id="category-group">
            <label>Categoría</label>
            <select id="category">
              <option value="Supermercado / Comida">Supermercado / Comida</option>
              <option value="Salidas / Ocio">Salidas / Ocio</option>
              <option value="Transporte">Transporte</option>
              <option value="Servicios / Cuentas">Servicios / Cuentas</option>
              <option value="Indumentaria">Indumentaria</option>
              <option value="Otros Gastos">Otros Gastos</option>
            </select>
          </div>
          <button type="submit" class="btn-submit">Guardar Movimiento</button>
        </form>
      </div>
    </div>

    <!-- PÁGINA 2: DESGLOSE Y DETALLE DE GASTOS -->
    <div id="page-history" class="page">
      <h3 style="margin-bottom: 14px; font-size: 1rem;">📋 Desglose Completo de Gastos</h3>
      <div id="history-container" class="history-list">
        <!-- Los ítems se cargan dinámicamente -->
      </div>
    </div>

  </div>

  <!-- NAVEGACIÓN INFERIOR (TABS) -->
  <nav class="bottom-nav">
    <div class="nav-item active" id="tab-home" onclick="switchPage('home')">
      <span class="nav-icon">📊</span>
      <span>Inicio</span>
    </div>
    <div class="nav-item" id="tab-history" onclick="switchPage('history')">
      <span class="nav-icon">📝</span>
      <span>Desglose</span>
    </div>
  </nav>

  <script>
    // ESTADO DE LA APLICACIÓN Y PERSISTENCIA (localStorage)
    let transactions = JSON.parse(localStorage.getItem('iol_expenses_app_data')) || [];
    let myChart = null;

    function saveToStorage() {
      localStorage.setItem('iol_expenses_app_data', JSON.stringify(transactions));
      updateApp();
    }

    function toggleCategoryInput() {
      const type = document.getElementById('type').value;
      const catGroup = document.getElementById('category-group');
      catGroup.style.display = (type === 'income') ? 'none' : 'block';
    }

    // REGISTRAR NUEVO MOVIMIENTO
    document.getElementById('finance-form').addEventListener('submit', function(e) {
      e.preventDefault();
      const type = document.getElementById('type').value;
      const amount = parseFloat(document.getElementById('amount').value);
      const category = type === 'income' ? 'Ingreso' : document.getElementById('category').value;

      if (!amount || amount <= 0) return;

      const newTransaction = {
        id: Date.now(),
        type: type,
        amount: amount,
        category: category,
        date: new Date().toLocaleDateString('es-AR', { day: '2-digit', month: '2-digit', year: 'numeric' })
      };

      transactions.push(newTransaction);
      saveToStorage();

      document.getElementById('amount').value = '';
    });

    // ELIMINAR REGISTRO
    function deleteItem(id) {
      transactions = transactions.filter(t => t.id !== id);
      saveToStorage();
    }

    // NAVEGACIÓN ENTRE PÁGINAS
    function switchPage(page) {
      document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
      document.querySelectorAll('.nav-item').forEach(t => t.classList.remove('active'));

      if (page === 'home') {
        document.getElementById('page-home').classList.add('active');
        document.getElementById('tab-home').classList.add('active');
      } else {
        document.getElementById('page-history').classList.add('active');
        document.getElementById('tab-history').classList.add('active');
      }
    }

    // RENDERIZAR Y ACTUALIZAR DATOS EN PANTALLA
    function updateApp() {
      let incomeTotal = transactions.filter(t => t.type === 'income').reduce((acc, t) => acc + t.amount, 0);
      let expenseTotal = transactions.filter(t => t.type === 'expense').reduce((acc, t) => acc + t.amount, 0);

      document.getElementById('total-income').innerText = `$${incomeTotal.toLocaleString('es-AR')}`;
      document.getElementById('total-expense').innerText = `$${expenseTotal.toLocaleString('es-AR')}`;

      // ACTUALIZAR GRÁFICO DE TORTA
      let remaining = incomeTotal > expenseTotal ? incomeTotal - expenseTotal : 0;
      let usedPercentage = incomeTotal > 0 ? Math.min(100, (expenseTotal / incomeTotal) * 100) : 0;

      const ctx = document.getElementById('budgetChart').getContext('2d');
      if (myChart) myChart.destroy();

      myChart = new Chart(ctx, {
        type: 'doughnut',
        data: {
          labels: ['Disponible', 'Gastado'],
          datasets: [{
            data: [remaining, expenseTotal],
            backgroundColor: ['#00C853', '#FF4D4D'],
            borderWidth: 2,
            borderColor: '#FFFFFF'
          }]
        },
        options: {
          cutout: '70%',
          plugins: {
            legend: { position: 'bottom' },
            tooltip: {
              callbacks: {
                label: function(context) {
                  return ` ${context.label}: $${context.raw.toLocaleString('es-AR')}`;
                }
              }
            }
          }
        }
      });

      // ACTUALIZAR LISTA DE DESGLOSE
      const historyContainer = document.getElementById('history-container');
      historyContainer.innerHTML = '';

      const expensesList = transactions.filter(t => t.type === 'expense').reverse();

      if (expensesList.length === 0) {
        historyContainer.innerHTML = '<div style="text-align:center; color:var(--text-muted); padding:20px;">No hay gastos registrados aún.</div>';
        return;
      }

      expensesList.forEach(t => {
        const item = document.createElement('div');
        item.className = 'history-item';
        item.innerHTML = `
          <div class="item-info">
            <span class="item-title">${t.category}</span>
            <span class="item-date">${t.date}</span>
          </div>
          <div style="display:flex; align-items:center;">
            <span class="item-amount text-danger">-$${t.amount.toLocaleString('es-AR')}</span>
            <button class="btn-delete" onclick="deleteItem(${t.id})">🗑️</button>
          </div>
        `;
        historyContainer.appendChild(item);
      });
    }

    // CARGA INICIAL
    window.onload = function() {
      updateApp();
    };
  </script>
</body>
</html>
