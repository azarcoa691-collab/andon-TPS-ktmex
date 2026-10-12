<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistema Andon - KTMEX TPS</title>
    <!-- Librerías para exportar a PDF -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.28/jspdf.plugin.autotable.min.js"></script>
    
    <!-- Firebase SDK (Compat) -->
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
    
    <style>
        :root {
            --bg-dark: #121212;
            --text-light: #f5f5f5;
            --panel-bg: #1e1e1e;
        }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-dark);
            color: var(--text-light);
            margin: 0;
            padding: 20px;
        }
        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-bottom: 2px solid #333;
            padding-bottom: 15px;
            margin-bottom: 20px;
        }
        .controls {
            display: flex;
            gap: 15px;
            align-items: center;
        }
        select, input[type="file"], button {
            padding: 10px;
            border-radius: 5px;
            border: 1px solid #444;
            background-color: #333;
            color: white;
            font-size: 14px;
        }
        button {
            cursor: pointer;
            font-weight: bold;
            transition: 0.3s;
        }
        button:hover { filter: brightness(1.2); }
        .btn-pdf { background-color: #007bff; border: none; }
        .btn-abort { background-color: #dc3545; border: none; padding: 5px 10px;}
        
        .dashboard-info {
            display: flex;
            justify-content: space-between;
            background: var(--panel-bg);
            padding: 15px;
            border-radius: 8px;
            margin-bottom: 20px;
            font-size: 18px;
        }
        .pattern-display { font-weight: bold; color: #f39c12; font-size: 22px; }
        
        table {
            width: 100%;
            border-collapse: collapse;
            background: var(--panel-bg);
            border-radius: 8px;
            overflow: hidden;
        }
        th, td {
            padding: 12px 15px;
            text-align: center;
            border-bottom: 1px solid #333;
        }
        th { background-color: #2c3e50; font-weight: bold; }
        .progress-bar {
            height: 10px;
            border-radius: 5px;
            background-color: #444;
            position: relative;
            margin-top: 5px;
        }
        .progress-fill {
            height: 100%;
            border-radius: 5px;
            transition: width 0.5s linear;
        }
        
        /* Modal Styles */
        .modal {
            display: none; position: fixed; z-index: 100;
            left: 0; top: 0; width: 100%; height: 100%;
            background-color: rgba(0,0,0,0.7);
        }
        .modal-content {
            background-color: var(--panel-bg);
            margin: 15% auto; padding: 20px;
            border: 1px solid #888; width: 350px; border-radius: 8px;
        }
        .modal-content input, .modal-content select {
            width: 90%; margin-bottom: 15px;
            background-color: #222; color: white; border: 1px solid #555; padding: 8px;
        }
        @keyframes blink {
            0% { opacity: 1; }
            100% { opacity: 0.3; }
        }
    </style>
</head>
<body>

    <div class="header">
        <h2>Andon - Control de Criticidad TPS</h2>
        <div class="controls">
            <select id="pressSelect">
                <option value="">Seleccionar Prensa...</option>
                <option value="LPS150">LPS150</option>
                <option value="TD200">TD200</option>
                <option value="TR1600 - TR1">TR1600 - TR1</option>
                <option value="TR800 - TR2">TR800 - TR2</option>
                <option value="TR1600 - TR3">TR1600 - TR3</option>
                <option value="TR800 - TR4">TR800 - TR4</option>
                <option value="PG300 - PG1">PG300 - PG1</option>
                <option value="PG200 - PG2">PG200 - PG2</option>
                <option value="PG300 - PG3">PG300 - PG3</option>
                <option value="PG200 - PG4">PG200 - PG4</option>
            </select>
            <input type="file" id="fileInput" accept=".csv">
            <button class="btn-pdf" onclick="descargarPDF()">📥 Descargar Reporte PDF</button>
        </div>
    </div>

    <div class="dashboard-info">
        <div>Prensa Activa: <strong id="activePressDisplay" style="color:#007bff;">Ninguna</strong></div>
        <div class="pattern-display">Patrón a Correr: <span id="currentPattern">--</span></div>
        <div>Último Censo: <span id="lastSyncTime">--:--</span></div>
    </div>

    <table>
        <thead>
            <tr>
                <th>Sebango(s)</th>
                <th>Process Name</th>
                <th>Horas de Alcance</th>
                <th>Estatus Visual</th>
                <th>Acción Rápida</th>
            </tr>
        </thead>
        <tbody id="tableBody">
            <!-- Los números de parte se inyectarán aquí -->
        </tbody>
    </table>

    <!-- Modal Aborto de Producción -->
    <div id="abortModal" class="modal">
        <div class="modal-content">
            <h3 style="margin-top:0;">Registrar Aborto</h3>
            <p>Sebango: <strong id="modalSebango" style="color:#f39c12;"></strong></p>
            <label>Piezas Producidas:</label>
            <input type="number" id="abortQty" min="0" placeholder="Ej. 1500">
            <label>Motivo de Aborto:</label>
            <select id="abortReason">
                <option value="Troquel">Problema de Troquel</option>
                <option value="Mala Operación">Mala Operación</option>
                <option value="Blank Defectuoso">Blank Defectuoso</option>
                <option value="Falta de Material">Falta de Material</option>
            </select>
            <div style="display:flex; justify-content:space-between; margin-top: 15px;">
                <button onclick="cerrarModal()" style="background:#555;">Cancelar</button>
                <button onclick="confirmarAborto()" class="btn-abort">Registrar y Avanzar</button>
            </div>
        </div>
    </div>

    <script>
        // ==========================================
        // 1. BASE DE DATOS DE CALENDARIOS COMPLETOS (OCT 2026)
        // ==========================================
        const calendariosOctubre = {
            "LPS150":       { 11: "D", 12: "A", 13: "B", 14: "A", 15: "B", 16: "A" },
            "TR800 - TR2":  { 11: "C", 12: "B", 13: "A", 14: "B", 15: "C", 16: "D" },
            "TR1600 - TR3": { 11: "C", 12: "B", 13: "A", 14: "B", 15: "A", 16: "B" },
            "TR800 - TR4":  { 11: "A", 12: "B", 13: "A", 14: "B", 15: "C", 16: "D" },
            "PG300 - PG1":  { 11: "G", 12: "B", 13: "C", 14: "D", 15: "E", 16: "F" },
            "PG200 - PG2":  { 11: "C", 12: "D", 13: "A", 14: "B", 15: "C", 16: "D" },
            "PG200 - PG4":  { 11: "B", 12: "C", 13: "D", 14: "E", 15: "B", 16: "C" },
            "TD200":        { 11: "E", 12: "B", 13: "A", 14: "B", 15: "A", 16: "B" },
            "PG300 - PG3":  { 11: "F", 12: "B", 13: "C", 14: "D", 15: "E", 16: "B" }
        };

        // ==========================================
        // 2. LÓGICA DE URGENCIA Y PATRONES
        // ==========================================
        function getUrgencia(horas) {
            if (horas > 17) return { c: "#28a745", e: "🧘✅", l: "Calma", p: 100 };
            if (horas > 8)  return { c: "#007bff", e: "⚙️ 🏃", l: "Acción", p: 60 };
            if (horas > 3)  return { c: "#fd7e14", e: "⚠️ ⚡", l: "Atención", p: 30 };
            return { c: "#dc3545", e: "🚨 🔥", l: horas <= 2 ? "CRÍTICO" : "URGENTE", p: 10, b: horas <= 2 };
        }

        function calcularPatron(prensa) {
            if (!prensa || !calendariosOctubre[prensa]) return "--";
            
            const now = new Date();
            const dia = now.getDate();
            const hora = now.getHours();
            
            // Turno 1: 06:00 a 17:59 | Turno 2: 18:00 a 05:59
            const turno = (hora >= 6 && hora < 18) ? 1 : 2;
            const letraPatron = calendariosOctubre[prensa][dia] || "?";
            
            return `${letraPatron}${turno}`;
        }

        // ==========================================
        // 3. VARIABLES Y PROCESAMIENTO
        // ==========================================
        let inventoryData = [];
        let syncTimestamp = Date.now();
        let selectedPress = "";
        let abortHistory = [];
        let currentAbortItem = null;

        document.getElementById('fileInput').addEventListener('change', function(e) {
            const file = e.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = function(event) {
                procesarCSV(event.target.result);
                syncTimestamp = Date.now();
                document.getElementById('lastSyncTime').innerText = new Date().toLocaleTimeString('es-MX', { hour12: false });
                renderTable();
            };
            reader.readAsText(file);
        });

        function procesarCSV(csvText) {
            const lines = csvText.split('\n');
            inventoryData = [];
            for (let i = 1; i < lines.length; i++) {
                if (lines[i].trim() === '') continue;
                const cols = lines[i].split(',');
                if (cols.length >= 3) {
                    inventoryData.push({
                        sebango: cols[0].trim(),
                        process: cols[1].trim(),
                        originalHours: parseFloat(cols[2].trim()),
                        aborted: false
                    });
                }
            }
        }

        document.getElementById('pressSelect').addEventListener('change', function(e) {
            selectedPress = e.target.value;
            document.getElementById('activePressDisplay').innerText = selectedPress || "Ninguna";
            document.getElementById('currentPattern').innerText = calcularPatron(selectedPress);
            renderTable();
        });

        function renderTable() {
            if (!selectedPress) return;
            const tbody = document.getElementById('tableBody');
            tbody.innerHTML = '';
            
            // Actualiza el patrón en caso de cambio de turno dinámico
            document.getElementById('currentPattern').innerText = calcularPatron(selectedPress);

            const horasTranscurridas = (Date.now() - syncTimestamp) / (1000 * 60 * 60);

            let filteredData = inventoryData
                .filter(item => item.process === selectedPress && !item.aborted)
                .map(item => ({ ...item, currentHours: Math.max(0, item.originalHours - horasTranscurridas) }))
                .sort((a, b) => a.currentHours - b.currentHours);

            filteredData.forEach(item => {
                const tr = document.createElement('tr');
                const urg = getUrgencia(item.currentHours);
                const blinkClass = urg.b ? 'style="animation: blink 1s infinite alternate;"' : '';
                
                tr.innerHTML = `
                    <td style="font-weight: bold; font-size: 16px;">${item.sebango}</td>
                    <td>${item.process}</td>
                    <td>
                        <div style="font-size: 18px; font-weight:bold; color:${urg.c}">
                            ${item.currentHours.toFixed(2)} Hrs
                        </div>
                        <div class="progress-bar">
                            <div class="progress-fill" style="width: ${urg.p}%; background-color: ${urg.c};"></div>
                        </div>
                    </td>
                    <td ${blinkClass}>
                        <span style="font-size: 22px;">${urg.e}</span><br>
                        <span style="color:${urg.c}; font-weight:bold;">${urg.l}</span>
                    </td>
                    <td>
                        <button class="btn-abort" onclick="abrirModal('${item.sebango}')">🛑 Abortar</button>
                    </td>
                `;
                tbody.appendChild(tr);
            });
        }

        setInterval(renderTable, 60000); // Refresca las horas cada minuto

        // ==========================================
        // 4. LÓGICA DE ABORTO Y PDF
        // ==========================================
        function abrirModal(sebango) {
            currentAbortItem = sebango;
            document.getElementById('modalSebango').innerText = sebango;
            document.getElementById('abortQty').value = '';
            document.getElementById('abortModal').style.display = 'block';
        }

        function cerrarModal() {
            document.getElementById('abortModal').style.display = 'none';
            currentAbortItem = null;
        }

        function confirmarAborto() {
            const qty = document.getElementById('abortQty').value;
            const reason = document.getElementById('abortReason').value;
            
            if (!qty) return alert("Ingrese la cantidad de piezas producidas.");

            abortHistory.push({
                sebango: currentAbortItem, piezas: qty, motivo: reason, fecha: new Date().toLocaleString('es-MX')
            });

            const itemIndex = inventoryData.findIndex(i => i.sebango === currentAbortItem);
            if (itemIndex > -1) inventoryData[itemIndex].aborted = true;

            cerrarModal();
            renderTable();
        }

        function descargarPDF() {
            if (abortHistory.length === 0) return alert("No hay abortos registrados.");

            const { jsPDF } = window.jspdf;
            const doc = new jsPDF();
            doc.setFontSize(14);
            doc.text(`Reporte de Producción Abortada - ${selectedPress}`, 14, 20);
            
            doc.autoTable({
                head: [["Sebango", "Piezas", "Motivo", "Fecha/Hora"]],
                body: abortHistory.map(r => [r.sebango, r.piezas, r.motivo, r.fecha]),
                startY: 30, theme: 'grid', headStyles: { fillColor: [44, 62, 80] }
            });

            doc.save(`Abortos_${selectedPress.replace(/\s+/g, '')}_${new Date().getTime()}.pdf`);
        }
    </script>
</body>
</html>
