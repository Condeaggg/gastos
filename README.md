<html lang="es">
<head>
  <meta charset="UTF-8" />
  <title>Administrador de Pagos (Local)</title>
  <style>
    :root {
      --fondo: #ecf1f2;
      --primario: #2d3c3c;
      --texto: #0a0b0c;
      --acento: #94c1c7;
      --secundario: #8a9da2;
    }

    body {
      font-family: 'Segoe UI', sans-serif;
      background-color: var(--fondo);
      color: var(--texto);
      padding: 2rem;
      margin: 0;
    }

    header {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .saldo-container {
      text-align: right;
    }

    button {
      background: var(--primario);
      color: white;
      padding: 0.5rem 1rem;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      font-weight: bold;
      transition: 0.2s;
    }

    button:hover {
      background: var(--acento);
    }

    .borrar {
      background-color: #dc3545;
      margin-left: 1rem;
    }

    .borrar:hover {
      background-color: #a71d2a;
    }

    ul {
      padding: 0;
      list-style: none;
    }

    li {
      background: white;
      padding: 1rem;
      margin-top: 1rem;
      border-radius: 10px;
      box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .modal {
      display: none;
      position: fixed;
      top: 0; left: 0;
      width: 100%; height: 100%;
      background: rgba(0,0,0,0.4);
      justify-content: center;
      align-items: center;
    }

    .modal-content {
      background: var(--secundario);
      padding: 2rem;
      border-radius: 12px;
      color: white;
      text-align: center;
      width: 300px;
    }

    input {
      width: 90%;
      padding: 0.5rem;
      margin-top: 0.5rem;
      border-radius: 6px;
      border: 1px solid #ccc;
    }
  </style>
</head>
<body>

  <header>
    <h1>Administrador de Pagos</h1>
    <div class="saldo-container">
      <button onclick="mostrarModalSaldo()">Saldo: $<span id="saldo">0.00</span></button>
      <div id="saldoMostrado"></div>
    </div>
  </header>

  <ul id="listaGastos"></ul>
  <button onclick="mostrarModalGasto()">+ Añadir Gasto</button>

  <!-- Modal Saldo -->
  <div class="modal" id="modalSaldo">
    <div class="modal-content">
      <h3>Agregar Saldo</h3>
      <input type="number" id="inputSaldo" placeholder="Ingrese el saldo" />
      <button onclick="agregarSaldo()">Enviar</button>
    </div>
  </div>

  <!-- Modal Gasto -->
  <div class="modal" id="modalGasto">
    <div class="modal-content">
      <h3>Nuevo Gasto</h3>
      <input type="text" id="descripcionGasto" placeholder="Descripción" />
      <input type="number" id="cantidadGasto" placeholder="Cantidad" />
      <button onclick="agregarGasto()">Enviar</button>
    </div>
  </div>

  <script>
    let saldo = parseFloat(localStorage.getItem('saldo')) || 0;
    let gastos = JSON.parse(localStorage.getItem('gastos')) || [];

    function actualizarVista() {
      document.getElementById('saldo').textContent = saldo.toFixed(2);
      document.getElementById('saldoMostrado').textContent = `Saldo actual: $${saldo.toFixed(2)}`;
      const lista = document.getElementById('listaGastos');
      lista.innerHTML = '';
      gastos.forEach((g, index) => {
        const li = document.createElement('li');
        const span = document.createElement('span');
        span.textContent = `${g.descripcion} - $${parseFloat(g.cantidad).toFixed(2)} (registrado el ${g.fecha})`;

        const btn = document.createElement('button');
        btn.textContent = 'Borrar';
        btn.className = 'borrar';
        btn.onclick = () => borrarGasto(index);

        li.appendChild(span);
        li.appendChild(btn);
        lista.appendChild(li);
      });
    }

    function mostrarModalSaldo() {
      document.getElementById('modalSaldo').style.display = 'flex';
    }

    function mostrarModalGasto() {
      document.getElementById('modalGasto').style.display = 'flex';
    }

    window.onclick = function (e) {
      if (e.target.classList.contains('modal')) {
        e.target.style.display = 'none';
      }
    };

    function agregarSaldo() {
      const valor = parseFloat(document.getElementById('inputSaldo').value);
      if (!isNaN(valor) && valor > 0) {
        saldo += valor;
        localStorage.setItem('saldo', saldo);
        actualizarVista();
      }
      document.getElementById('modalSaldo').style.display = 'none';
      document.getElementById('inputSaldo').value = '';
    }

    function agregarGasto() {
      const desc = document.getElementById('descripcionGasto').value.trim();
      const cant = parseFloat(document.getElementById('cantidadGasto').value);
      const fecha = new Date().toLocaleDateString('es-ES');

      if (desc && !isNaN(cant) && cant > 0) {
        gastos.push({ descripcion: desc, cantidad: cant, fecha });
        saldo -= cant;
        localStorage.setItem('gastos', JSON.stringify(gastos));
        localStorage.setItem('saldo', saldo);
        actualizarVista();
      }

      document.getElementById('modalGasto').style.display = 'none';
      document.getElementById('descripcionGasto').value = '';
      document.getElementById('cantidadGasto').value = '';
    }

    function borrarGasto(index) {
      const cantidad = parseFloat(gastos[index].cantidad);
      saldo += cantidad;
      gastos.splice(index, 1);
      localStorage.setItem('gastos', JSON.stringify(gastos));
      localStorage.setItem('saldo', saldo);
      actualizarVista();
    }

    window.onload = actualizarVista;
  </script>

</body>
</html>
