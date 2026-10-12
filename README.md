<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Andon de Estampado</title>
    <!-- Librerías para exportar a PDF -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.28/jspdf.plugin.autotable.min.js"></script>
    
    <!-- Firebase SDK (Compat) -->
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
    
    <style>
        :root {
            --bg-color: #f0f2f5;
            --text-dark: #333;
            --header-blue: #4285f4;
            --color-critico: #dc3545;
            --color-atencion: #fd7e14;
            --color-accion: #007bff;
            --color-calma: #28a745;
        }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-dark);
            margin: 0;
            padding: 0;
            height: 100vh;
            display: flex;
            flex-direction: column;
        }
        
        /* HEADER Y TOP BAR */
        .top-bar {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 10px 20px;
            background-color: #ffffff;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05);
        }
        .title-container {
            display: flex;
            flex-direction: column;
        }
        .main-title {
            background-color: var(--header-blue);
            color: white;
            padding: 5px 15px;
            font-size: 24px;
            font-weight: bold;
            border-radius: 4px;
            display: inline-block;
            margin-bottom: 5px;
        }
        .current-time {
            font-size: 14px;
            color: #666;
        }
        .top-buttons button {
            background-color: #202124;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 20px;
            font-weight: bold;
            cursor: pointer;
            margin-left: 10px;
            font-size: 12px;
            display: inline-flex;
            align-items: center;
            gap: 5px;
        }
        .top-buttons button:hover { background-color: #3c4043; }
        #fileInput { display: none; } /* Ocultar el input nativo */

        /* CONTENEDOR PRINCIPAL */
        .main-content {
            flex: 1;
            padding: 20px;
            overflow-y: auto;
        }

        /* VISTA 1: DASHBOARD (TARJETAS) */
        .dashboard {
            display: flex;
            gap: 15px;
            height: 100%;
        }
        .card {
            background: white;
            border-radius: 20px;
            flex: 1;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            box-shadow: 0 4px 6px rgba(0,0,0,0.05);
            border-bottom: 8px solid #ccc;
            cursor: pointer;
            transition: transform 0.2s, box-shadow 0.2s;
            text-align: center;
            padding: 20px 10px;
        }
        .card:hover { transform: translateY(-5px); box-shadow: 0 8px 15px rgba(0,0,0,0.1); }
        .card-title { font-size: 22px; font-weight: bold; margin-bottom: 5px; }
        .card-subtitle { font-size: 12px; color: #888; margin-bottom: auto; }
        .card-icon { font-size: 40px; margin: 20px 0; }
        .card-status { font-weight: bold; font-size: 16px; text-transform: uppercase; }

        /* Clases de colores dinámicos */
        .critico { color: var(--color-critico); border-color: var(--color-critico); }
        .atencion { color: var(--color-atencion); border-color: var(--color-atencion); }
        .accion { color: var(--color-accion); border-color: var(--color-accion); }
        .calma { color: var(--color-calma); border-color: var(--color-calma); }

        /* VISTA 2: DETALLES (TABLA) */
        .detail-view { display: none; }
        .detail-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
            background: white;
            padding: 15px 20px;
            border-radius: 10px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05);
        }
        .btn-back {
            background-color: #e0e0e0; border: none; padding: 10px 15px; border-radius: 5px;
            font-weight: bold; cursor: pointer; color: #333;
        }
        .btn-pdf {
            background-color: #007bff; border: none; padding: 10px 15px; border-radius: 5px;
            font-weight: bold; cursor: pointer; color: white;
        }
        .btn-abort { background-color: #dc3545; color: white; border: none; padding: 8px 12px; border-radius: 5px; cursor: pointer; font-weight: bold; }
        
        table { width: 100%; border-collapse: collapse; background: white; border-radius: 10px; overflow: hidden; box-shadow: 0 2px 4px rgba(0,0,0,0.05); }
        th, td { padding: 12px 15px; text-align: center; border-bottom: 1px solid #eee; }
        th { background-color: #f8f9fa; font-weight: bold; color: #555; }
        
        .progress-bar { height: 10px; border-radius: 5px; background-color: #e9ecef; position: relative; margin-top: 5px; }
        .progress-fill { height: 100%; border-radius: 5px; transition: width 0.5s linear; }

        /* FOOTER */
        .footer {
            text-align: center;
            padding: 10px;
            font-size: 12px;
            color: #888;
            background-color: white;
            border-top: 1px solid #eee;
        }

        /* MODAL */
        .modal { display: none; position: fixed; z-index: 100; left: 0; top: 0; width: 100%; height: 100%; background-color: rgba(0,0,0,0.5); }
        .modal-content { background-color: white; margin: 15% auto; padding: 20px; border-radius: 10px; width: 350px; box-shadow: 0 5px 15px rgba(0,0,0,0.3); }
        .modal-content input, .modal-content select { width: 90%; margin-bottom: 15px; padding: 8px; border: 1px solid #ccc; border-radius: 4px; }
        
        @keyframes blink { 0% { opacity: 1; } 100% { opacity: 0.3; } }
    </style>
</head>
<body>

    <!-- HEADER BAR -->
    <div class="top-bar">
        <div class="title-container">
            <span class="main-title">🚨 Andon de Estampado - Selecciona prensa</span>
            <span class="current-time" id="realTimeClock">Cargando hora...</span>
        </div>
        <div class="top-buttons">
            <button onclick="toggleFullScreen()">📺 PANTALLA COMPLETA</button>
            <button onclick="document.getElementById('fileInput').click()">📁 CARGAR CSV</button>
            <input type="file" id="fileInput" accept=".csv">
        </div>
    </div>

    <!-- CONTENEDOR PRINCIPAL -->
    <div class="main-content">
        
        <!-- VISTA 1: DASHBOARD DE TARJETAS -->
        <div class="dashboard" id="dashboardView">
            <!-- Las tarjetas se generan dinámicamente con JS -->
        </div>

        <!-- VISTA 2: DETALLES DE PRENSA (TABLA) -->
        <div class="detail-view" id="detailView">
            <div class="detail-header">
                <button class="btn-back" onclick="mostrarDashboard()">⬅ Volver a Prensa</button>
                <div style="text-align:center;">
                    <h2 style="margin:0; color:var(--header-blue);" id="activePressDisplay">PRENSA</h2>
                    <strong style="color:#f39c12; font-size: 18px;">Patrón: <span id="currentPattern">--</span></strong>
                </div>
                <button class="btn-pdf" onclick="descargarPDF()">📥 Descargar Reporte PDF</button>
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
                <tbody id="tableBody"></tbody>
            </table>
        </div>

    </div>

    <!-- FOOTER -->
    <div class="footer" id="footerText">
        Última carga de archivo: --:-- • Esperando archivo CSV
    </div>

    <!-- MODAL DE ABORTO -->
    <div id="abortModal" class="modal">
        <div class="modal-content">
            <h3 style="margin-top:0; color:var(--text-dark);">Registrar Aborto</h3>
            <p>Sebango: <strong id="modalSebango" style="color:var(--color-critico);"></strong></p>
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
                <button onclick="cerrarModal()" class="btn-back">Cancelar</button>
                <button onclick="confirmarAborto()" class="btn-abort">Registrar y Avanzar</button>
            </div>
        </div>
    </div>

    <script>
        // ==========================================
        // CONFIGURACIÓN DE PRENSAS Y CALENDARIOS
        // ==========================================
        const listaPrensas = [
            "LPS150", "TD200", "TR1600 - TR1", "TR800 - TR2", "TR1600 - TR3", 
            "TR800 - TR4", "PG300 - PG1", "PG200 - PG2", "PG300 - PG3", "PG200 - PG4"
        ];

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
        // VARIABLES GLOBALES
        // ==========================================
        let inventoryData = [];
        let syncTimestamp = Date.now();
        let currentFileName = "INVENTARIO_GENERAL.csv";
        let activePress = null;
        let abortHistory = [];
        let currentAbortItem = null;

        // ==========================================
        // RELOJ EN TIEMPO REAL
        // ==========================================
        function updateClock() {
            const now = new Date();
            const options = { year: 'numeric', month: '2-digit', day: '2-digit', hour: 'numeric', minute: '2-digit', second: '2-digit', hour12: true };
            document.getElementById('realTimeClock').innerText = now.toLocaleString('es-MX', options);
        }
        setInterval(updateClock, 1000);
        updateClock();

        // ==========================================
        // LÓGICA DE PROCESAMIENTO
        // ==========================================
        document.getElementById('fileInput').addEventListener('change', function(e) {
            const file = e.target.files[0];
            if (!file) return;
            currentFileName = file.name;
            const reader = new FileReader();
            reader.onload = function(event) {
                procesarCSV(event.target.result);
                syncTimestamp = Date.now();
                actualizarFooter();
                renderAll();
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

        function actualizarFooter() {
            const dateStr = new Date(syncTimestamp).toLocaleString('es-MX', { year: 'numeric', month: '2-digit', day: '2-digit', hour: 'numeric', minute: '2-digit', second: '2-digit', hour12: true });
            document.getElementById('footerText').innerText = `Última carga de archivo: ${dateStr} • ${currentFileName}`;
        }

        // ==========================================
        // RENDERIZADO DEL DASHBOARD (TARJETAS)
        // ==========================================
        function calcularStatusPrensa(prensa) {
            let items = inventoryData.filter(i => i.process === prensa && !i.aborted);
            let count = items.length;
            if (count === 0) return { count: 0, status: 'CALMA', clase: 'calma', icono: '✅', texto: 'CALMA' };

            let horasTranscurridas = (Date.now() - syncTimestamp) / 3600000;
            let criticos = 0, atencion = 0, accion = 0;

            items.forEach(i => {
                let h = Math.max(0, i.originalHours - horasTranscurridas);
                if (h <= 2) criticos++;
                else if (h <= 3) atencion++;
                else if (h <= 8) accion++;
            });

            if (criticos > 0) return { count, status: 'CRITICO', clase: 'critico', icono: '🚨', texto: `${criticos} CRÍTICO${criticos > 1 ? 'S' : ''}` };
            if (atencion > 0) return { count, status: 'ATENCION', clase: 'atencion', icono: '⚠️', texto: 'ATENCIÓN' };
            if (accion > 0) return { count, status: 'ACCION', clase: 'accion', icono: '⚙️', texto: 'ACCIÓN' };
            return { count, status: 'CALMA', clase: 'calma', icono: '🧘', texto: 'CALMA' };
        }

        function renderDashboard() {
            const container = document.getElementById('dashboardView');
            container.innerHTML = '';

            listaPrensas.forEach(prensa => {
                const stats = calcularStatusPrensa(prensa);
                const card = document.createElement('div');
                card.className = `card ${stats.clase}`;
                card.onclick = () => abrirDetalle(prensa);
                
                card.innerHTML = `
                    <div class="card-title ${stats.clase}">${prensa}</div>
                    <div class="card-subtitle">${stats.count} sebangos</div>
                    <div class="card-icon">${stats.icono}</div>
                    <div class="card-status ${stats.clase}">${stats.texto}</div>
                `;
                container.appendChild(card);
            });
        }

        // ==========================================
        // NAVEGACIÓN Y DETALLES (TABLA)
        // ==========================================
        function abrirDetalle(prensa) {
            activePress = prensa;
            document.getElementById('dashboardView').style.display = 'none';
            document.getElementById('detailView').style.display = 'block';
            document.getElementById('activePressDisplay').innerText = prensa;
            renderTabla();
        }

        function mostrarDashboard() {
            activePress = null;
            document.getElementById('detailView').style.display = 'none';
            document.getElementById('dashboardView').style.display = 'flex';
            renderDashboard();
        }

        function calcularPatron(prensa) {
            if (!prensa || !calendariosOctubre[prensa]) return "--";
            const now = new Date();
            const turno = (now.getHours() >= 6 && now.getHours() < 18) ? 1 : 2;
            return `${calendariosOctubre[prensa][now.getDate()] || "?"}${turno}`;
        }

        function renderTabla() {
            if (!activePress) return;
            document.getElementById('currentPattern').innerText = calcularPatron(activePress);
            
            const tbody = document.getElementById('tableBody');
            tbody.innerHTML = '';
            
            let horasTranscurridas = (Date.now() - syncTimestamp) / 3600000;
            
            let data = inventoryData
                .filter(i => i.process === activePress && !i.aborted)
                .map(i => ({ ...i, curH: Math.max(0, i.originalHours - horasTranscurridas) }))
                .sort((a, b) => a.curH - b.curH);

            data.forEach(item => {
                let urg = { c: "#28a745", e: "🧘✅", l: "Calma", p: 100, b: false };
                if (item.curH <= 8) urg = { c: "#007bff", e: "⚙️ 🏃", l: "Acción", p: 60, b: false };
                if (item.curH <= 3) urg = { c: "#fd7e14", e: "⚠️ ⚡", l: "Atención", p: 30, b: false };
                if (item.curH <= 2) urg = { c: "#dc3545", e: "🚨 🔥", l: "CRÍTICO", p: 10, b: true };

                const blink = urg.b ? 'style="animation: blink 1s infinite alternate;"' : '';
                
                tbody.innerHTML += `
                    <tr>
                        <td style="font-weight: bold; font-size: 14px;">${item.sebango}</td>
                        <td style="font-size: 12px;">${item.process}</td>
                        <td>
                            <div style="font-size: 16px; font-weight:bold; color:${urg.c}">${item.curH.toFixed(2)} Hrs</div>
                            <div class="progress-bar"><div class="progress-fill" style="width:${urg.p}%; background:${urg.c};"></div></div>
                        </td>
                        <td ${blink}>
                            <span style="font-size: 20px;">${urg.e}</span><br>
                            <span style="color:${urg.c}; font-weight:bold; font-size:12px;">${urg.l}</span>
                        </td>
                        <td><button class="btn-abort" onclick="abrirModal('${item.sebango}')">🛑 Abortar</button></td>
                    </tr>
                `;
            });
        }

        // Actualización cíclica
        function renderAll() {
            if (activePress) renderTabla();
            else renderDashboard();
        }
        setInterval(renderAll, 60000);
        renderDashboard(); // Render inicial vacío

        // ==========================================
        // ABORTO Y PDF
        // ==========================================
        function abrirModal(sebango) {
            currentAbortItem = sebango;
            document.getElementById('modalSebango').innerText = sebango;
            document.getElementById('abortQty').value = '';
            document.getElementById('abortModal').style.display = 'block';
        }
        function cerrarModal() { document.getElementById('abortModal').style.display = 'none'; currentAbortItem = null; }
        
        function confirmarAborto() {
            const qty = document.getElementById('abortQty').value;
            const reason = document.getElementById('abortReason').value;
            if (!qty) return alert("Ingrese la cantidad producida.");
            
            abortHistory.push({ sebango: currentAbortItem, piezas: qty, motivo: reason, fecha: new Date().toLocaleString('es-MX') });
            const idx = inventoryData.findIndex(i => i.sebango === currentAbortItem);
            if (idx > -1) inventoryData[idx].aborted = true;
            
            cerrarModal();
            renderAll();
        }

        function descargarPDF() {
            if (abortHistory.length === 0) return alert("No hay abortos registrados.");
            const doc = new window.jspdf.jsPDF();
            doc.text(`Reporte Abortos - ${activePress}`, 14, 20);
            doc.autoTable({
                head: [["Sebango", "Piezas", "Motivo", "Fecha"]],
                body: abortHistory.filter(r => inventoryData.find(i=>i.sebango===r.sebango)?.process === activePress).map(r => [r.sebango, r.piezas, r.motivo, r.fecha]),
                startY: 30, theme: 'grid'
            });
            doc.save(`Abortos_${activePress.replace(/\s/g,'')}.pdf`);
        }

        function toggleFullScreen() {
            if (!document.fullscreenElement) document.documentElement.requestFullscreen();
            else if (document.exitFullscreen) document.exitFullscreen();
        }
    </script>
</body>
</html>
