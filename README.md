<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Administrador de Pagos</title>
  <style>
    :root {
      --fondo: #ecf1f2;
      --primario: #2d3c3c;
      --texto: #0a0b0c;
      --acento: #94c1c7;
      --secundario: #8a9da2;
    }

    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      margin: 0;
      padding: 2rem;
      background: var(--fondo);
      color: var(--texto);
    }
    header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 2rem;
    }
    .saldo-container {
      text-align: right;
    }
    button {
      padding: 0.5rem 1rem;
      border: none;
      border-radius: 8px;
      background-color: var(--primario);
      color: white;
      cursor: pointer;
      font-weight: 500;
      transition: background-color 0.3s;
    }
    button:hover {
      background-color: var(--acento);
    }
    .borrar {
      background-color: #dc3545;
      margin-left: 1rem;
    }
    .borrar:hover {
      background-color: #a71d2a;
    }
    ul {
      list-style: none;
      padding: 0;
    }
    li {
      background: white;
      padding: 1rem;
      margin-bottom: 1rem;
      border-radius: 10px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.05);
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .modal {
      display: none;
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(0, 0, 0, 0.5);
      justify-content: center;
      align-items: center;
      z-index: 1000;
    }
    .modal-content {
      background: var(--secundario);
      padding: 2rem;
      border-radius: 12px;
      width: 320px;
      text-align: center;
      box-shadow: 0 4px 12px rgba(0,0,0,0.1);
      color: white;
    }
    input {
      margin: 0.5rem 0;
      padding: 0.6rem;
      width: 100%;
      border: 1px solid #ced4da;
      border-radius: 6px;
      font-size: 1rem;
    }
  </style>
</head>
<body>

  <header>
    <h1>Administrador de Pagos</h1>
    <div class="saldo-container">
      <button onclick="mostrarModalSaldo()">Saldo: $<span id="saldo">0</span></button>
      <div id="saldoMostrado"></div>
    </div>
  </header>

  <ul id="listaGastos"></ul>

  <button onclick="mostrarModalGasto()">+ Añadir Gasto</button>

  <!-- Modal para Saldo -->
  <div class="modal" id="modalSaldo">
    <div class="modal-content">
      <h3>Agregar Saldo</h3>
      <input type="number" id="inputSaldo" placeholder="Ingrese el saldo">
      <button onclick="agregarSaldo()">Enviar</button>
    </div>
  </div>

  <!-- Modal para Gasto -->
  <div class="modal" id="modalGasto">
    <div class="modal-content">
      <h3>Nuevo Gasto</h3>
      <input type="text" id="descripcionGasto" placeholder="Descripción">
      <input type="number" id="cantidadGasto" placeholder="Cantidad">
      <button onclick="agregarGasto()">Enviar</button>
    </div>
  </div>

  <!-- Firebase -->
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-database-compat.js"></script>
  <script>
    const firebaseConfig = {
      apiKey: "AlzaSybDmoEC4Y2-JHlAod_66ai7bRJtL8PhWT8",
      authDomain: "pagos-6aa03b.firebaseapp.com",
      databaseURL: "https://pagos-6aa03b-default-rtdb.firebaseio.com",
      projectId: "pagos-6aa03b",
      storageBucket: "pagos-6aa03b.appspot.com",
      messagingSenderId: "185112049813",
      appId: "1:185112049813:web:da79d567069c9dca1ea432"
    };

    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();

    let saldo = 0;

    window.onload = function() {
      db.ref("saldo").on("value", snapshot => {
        saldo = snapshot.val() || 0;
        document.getElementById('saldo').textContent = saldo.toFixed(2);
        document.getElementById('saldoMostrado').textContent = `Saldo actual: $${saldo.toFixed(2)}`;
      });

      db.ref("gastos").on("value", snapshot => {
        const lista = document.getElementById("listaGastos");
        lista.innerHTML = "";
        const gastos = snapshot.val();
        if (gastos) {
          Object.entries(gastos).forEach(([id, { descripcion, cantidad, fecha }]) => {
            const li = document.createElement("li");
            const texto = document.createElement("span");
            texto.textContent = `${descripcion} - $${parseFloat(cantidad).toFixed(2)} (registrado el ${fecha})`;
            const botonBorrar = document.createElement("button");
            botonBorrar.textContent = "Borrar";
            botonBorrar.className = "borrar";
            botonBorrar.onclick = function() {
              db.ref(`gastos/${id}`).remove();
              db.ref("saldo").set(saldo + parseFloat(cantidad));
            };
            li.appendChild(texto);
            li.appendChild(botonBorrar);
            lista.appendChild(li);
          });
        }
      });
    }

    function mostrarModalSaldo() {
      document.getElementById('modalSaldo').style.display = 'flex';
    }

    function mostrarModalGasto() {
      document.getElementById('modalGasto').style.display = 'flex';
    }

    function agregarSaldo() {
      const cantidad = parseFloat(document.getElementById('inputSaldo').value);
      if (!isNaN(cantidad)) {
        db.ref("saldo").set(saldo + cantidad);
      }
      document.getElementById('modalSaldo').style.display = 'none';
      document.getElementById('inputSaldo').value = '';
    }

    function agregarGasto() {
      const descripcion = document.getElementById('descripcionGasto').value.trim();
      const cantidad = parseFloat(document.getElementById('cantidadGasto').value);
      const fecha = new Date().toLocaleDateString('es-ES');

      if (descripcion && !isNaN(cantidad)) {
        const nuevoGasto = db.ref("gastos").push();
        nuevoGasto.set({ descripcion, cantidad, fecha });
        db.ref("saldo").set(saldo - cantidad);
      }

      document.getElementById('modalGasto').style.display = 'none';
      document.getElementById('descripcionGasto').value = '';
      document.getElementById('cantidadGasto').value = '';
    }

    window.onclick = function(event) {
      if (event.target.classList.contains('modal')) {
        event.target.style.display = "none";
      }
    }
  </script>
</body>
</html>
