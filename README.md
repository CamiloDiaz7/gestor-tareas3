<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Gestor de Tareas</title>
    <style>
        :root {
            --primary-color: #4A90E2;
            --secondary-color: #50E3C2;
            --background-color: #f7f9fc;
            --header-bg-color: #2c3e50;
            --header-text-color: #ffffff;
            --border-color: #d1d9e6;
        }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 20px;
            background-image: url('pexels-raybilcliff-2548493.jpg');
            background-size: cover;
            background-position: center;
            background-attachment: fixed;
            background-repeat: no-repeat;
            width: 75%;
            margin-left: 0;
            padding-left: 12%;
            height: 2px;
        }    

        button {
            margin: 10px 0;
            padding: 10px 20px;
            background-color: var(--primary-color);
            color: #fff;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
        }
        button:hover {
            background-color: var(--secondary-color);
        }
        table {
            width: 100%;
            border-collapse: collapse;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
            background: transparent;
            border-radius: 10px;
            overflow: hidden;
        }
        th, td {
            border: 1px solid rgba(255, 255, 255, 0.3);
            padding: 5px 10px;
            text-align: left;
            transition: background 0.3s, box-shadow 0.3s;
        }
        th {
            background-color: rgba(44, 62, 80, 0.9);
            color: var(--header-text-color);
            font-weight: 600;
            cursor: pointer;
        }
        tr:hover td {
            background-color: rgba(255, 255, 255, 0.1);
            box-shadow: 0 2px 10px rgba(0,0,0,0.2);
        }
        th:nth-child(1), td:nth-child(1) { width: 6%; }
        th:nth-child(2), td:nth-child(2) { width: 0%; }
        th:nth-child(3), td:nth-child(3) { width: 15%; }
        th:nth-child(4), td:nth-child(4) { width: 10%; }
        th:nth-child(5), td:nth-child(5) { width: 5%; }
        th:nth-child(6), td:nth-child(6) { width: 0%; }
        th:nth-child(7), td:nth-child(7) { width: 3%; }
        th:nth-child(8), td:nth-child(8) { width: 35%; }
        
        .rojo { background-color: rgba(255, 0, 0, 0.4); }
        .naranja { background-color: rgba(255, 165, 0, 0.4); }
        .verde { background-color: rgba(0, 128, 0, 0.4); }
        .azul { background-color: transparent; }
        .gris { background-color: transparent; }
        input[type="date"], input[type="text"] {
            width: 100%;
            box-sizing: border-box;
            padding: 6px;
            border: 1px solid var(--border-color);
            border-radius: 4px;
            background: black;
            color: #ffffff;
            font-family: cursive;
        }
    </style>
</head>
<body>
    <div class="container">
    <h1>Gestor de Tareas</h1>

    <button id="btnAdd">Agregar Nueva Fila</button>

    <table>
        <thead>
            <tr>
                <th onclick="sortTableByDate(0)">Fecha de Entrada</th>
                <th onclick="sortTableByDate(1)">Fecha de Follow Up</th>
                <th>Nombre</th>
                <th>Liability BI</th>
                <th>UM</th>
                <th>Balance</th>
                <th>Priority</th>
                <th>Nota</th>
            </tr>
        </thead>
        <tbody id="tablaBody"></tbody>
    </table>

    <script>
        let tareas = [];

        function loadTasks() {
            const saved = localStorage.getItem('tareas');
            tareas = saved ? JSON.parse(saved) : [];
            tareas.forEach((t, i) => appendRow(t, i));
            createBlankRow();
        }

        function saveTasks() {
            localStorage.setItem('tareas', JSON.stringify(tareas));
        }

        function appendRow(tarea, index) {
            const tbody = document.getElementById('tablaBody');
            const tr = document.createElement('tr');
            tr.dataset.index = index;

            const fields = ['fechaEntrada', 'fechaFollowUp', 'nombre', 'liabilityBI', 'um', 'balance', 'estado', 'nota'];

            fields.forEach(field => {
                const td = document.createElement('td');
                if (field === 'fechaEntrada') {
                    td.textContent = tarea[field];
                } else {
                    const input = document.createElement('input');
                    input.type = field.includes('fecha') ? 'date' : 'text';
                    input.value = tarea[field] || '';
                    input.dataset.field = field;
                    input.onchange = () => handleChange(input);
                    td.appendChild(input);
                }
                tr.appendChild(td);
            });

            applyColor(tr, tarea);
            tbody.appendChild(tr);
        }

        function handleChange(input) {
            const tr = input.closest('tr');
            const idx = parseInt(tr.dataset.index, 10);
            const field = input.dataset.field;

            tareas[idx][field] = input.value;
            saveTasks();
            applyColor(tr, tareas[idx]);
        }

        function applyColor(tr, tarea) {
            const hoy = new Date();
            const followUp = new Date(tarea.fechaFollowUp);
            tr.className = '';

            if (!tarea.fechaFollowUp) {
                tr.classList.add('gris');
            } else if (followUp < hoy) {
                tr.classList.add('rojo');
            } else if (followUp.toDateString() === hoy.toDateString()) {
                tr.classList.add('naranja');
            } else {
                tr.classList.add('verde');
            }
        }

        function createBlankRow() {
            const nuevaTarea = {
                fechaEntrada: new Date().toISOString().split('T')[0],
                fechaFollowUp: '',
                nombre: '',
                liabilityBI: '',
                um: '',
                balance: '',
                estado: '',
                nota: ''
            };
            tareas.push(nuevaTarea);
            appendRow(nuevaTarea, tareas.length - 1);
            saveTasks();
        }

        document.getElementById('btnAdd').addEventListener('click', createBlankRow);

        function sortTableByDate(colIndex) {
            const key = colIndex === 0 ? 'fechaEntrada' : 'fechaFollowUp';
            tareas.sort((a, b) => new Date(a[key]) - new Date(b[key]));
            const tbody = document.getElementById('tablaBody');
            tbody.innerHTML = '';
            tareas.forEach((t, i) => appendRow(t, i));
        }

        window.onload = loadTasks;
    </script>
</body>
</html>
