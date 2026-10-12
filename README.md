<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
    <title>ANDON Estampado TPS</title>
    
    <!-- Librerías para exportar a PDF y Firebase -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.28/jspdf.plugin.autotable.min.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

<style>
:root{--bg:#f0f2f6;--tx:#212529;--mut:#6c757d;--C:#dc3545;--A:#fd7e14;--X:#0d6efd;--K:#28a745;--b:#0d6efd;--dk:#212529}
*{box-sizing:border-box}html,body{height:100%;margin:0}
body{background:var(--bg);color:var(--tx);font-family:"Trebuchet MS","Segoe UI",system-ui,sans-serif;display:flex;flex-direction:column;padding:env(safe-area-inset-top,0px) env(safe-area-inset-right,0px) env(safe-area-inset-bottom,0px) env(safe-area-inset-left,0px);overflow:hidden}
header{display:flex;align-items:center;gap:10px;padding:8px 12px;flex:none;background:#fff;box-shadow:0 2px 5px rgba(0,0,0,0.05)}
.tt{flex:1;min-width:0}h1{margin:0;font-size:clamp(1.1rem,3.2vh,1.9rem);font-weight:800;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.clk{color:var(--mut);font-weight:700;font-size:clamp(.7rem,1.8vh,.95rem)}
.btns{display:flex;gap:6px;flex:none}
.btn{border:0;border-radius:30px;background:var(--dk);color:#fff;font-weight:800;font-size:clamp(.7rem,1.8vh,.9rem);padding:9px 14px;cursor:pointer;font-family:inherit;box-shadow:0 4px 12px rgba(0,0,0,.18);white-space:nowrap;display:flex;align-items:center;gap:5px}
.btn.blue{background:var(--b)}.btn.red{background:var(--C)}input[type=file]{display:none}
@media(max-width:560px){.btn span{display:none}.btn{padding:9px 11px}}
.card{background:#fff;border-radius:22px;box-shadow:0 10px 28px rgba(33,37,41,.12);margin:12px;padding:10px;flex:1;min-height:0;display:flex;flex-direction:column}
.hid{display:none!important}
.grid{flex:1;min-height:0;display:grid;grid-template-columns:repeat(auto-fit,minmax(150px,1fr));grid-auto-rows:1fr;gap:10px}
.press{border:0;border-bottom:6px solid;border-radius:18px;background:#f8f9fb;cursor:pointer;font-family:inherit;color:var(--tx);display:flex;flex-direction:column;align-items:center;justify-content:center;padding:6px;transition:transform 0.2s}
.press:hover{transform:translateY(-3px)}
.press b{font-size:clamp(1rem,3vh,1.5rem)}.press small{color:var(--mut);font-weight:700}.press em{font-style:normal;font-size:clamp(1.4rem,5vh,2.6rem)}.press strong{font-size:clamp(.7rem,1.8vh,.95rem)}
.sum{display:grid;grid-template-columns:repeat(4,1fr);gap:8px;margin-bottom:8px;flex:none}
.sum div{text-align:center;border-radius:14px;padding:5px 2px;background:#f8f9fb;font-weight:800}.sum span{display:block;font-size:clamp(1.2rem,4vh,2rem);line-height:1.1}.sum small{font-size:clamp(.55rem,1.5vh,.8rem);letter-spacing:.3px}
.hd{display:grid;grid-template-columns:1fr 1fr 1fr 40px;color:var(--mut);font-weight:800;font-size:.7rem;text-transform:uppercase;padding:0 14px 4px;flex:none}.hd :nth-child(2){text-align:center}.hd :nth-child(3){text-align:right}
#list{flex:1;min-height:0;display:grid;grid-auto-flow:column;gap:6px;overflow:auto}
.row{position:relative;display:grid;grid-template-columns:1fr 1fr 1fr 40px;align-items:center;background:#f8f9fb;border-radius:calc(var(--rh)*.25);border-left:calc(var(--rh)*.1) solid;padding:0 calc(var(--rh)*.2) calc(var(--rh)*.1);overflow:hidden;min-width:0}
.s{font-size:max(15px,calc(var(--rh)*.36));font-weight:800}.p{font-size:max(11px,calc(var(--rh)*.2));color:var(--mut);font-weight:700;text-align:center}
.h{text-align:right;font-size:max(17px,calc(var(--rh)*.42));font-weight:900;line-height:1;font-variant-numeric:tabular-nums;padding-right:10px}
.h small{display:block;font-size:max(9px,calc(var(--rh)*.15));letter-spacing:.5px;margin-top:2px}
.bar{position:absolute;left:calc(var(--rh)*.2);right:calc(var(--rh)*.2);bottom:calc(var(--rh)*.08);height:max(4px,calc(var(--rh)*.07));background:#e5e5e5;border-radius:6px;overflow:hidden}.bar i{display:block;height:100%;border-radius:6px;transition:width 0.5s}
.C{border-color:var(--C);color:var(--C)}.A{border-color:var(--A);color:var(--A)}.X{border-color:var(--X);color:var(--X)}.K{border-color:var(--K);color:var(--K)}
.bgC{background:var(--C)}.bgA{background:var(--A)}.bgX{background:var(--X)}.bgK{background:var(--K)}
.row .s{color:var(--tx)}.row.C{background:#fdecee;animation:bl 1.2s infinite}@keyframes bl{50%{opacity:.6}}
.foot{flex:none;text-align:center;color:var(--mut);font-style:italic;font-size:clamp(.6rem,1.5vh,.8rem);padding:5px}

/* Estilos para botón aborto y modal */
.btn-abort-row { background:none; border:none; cursor:pointer; font-size:1.2rem; display:flex; align-items:center; justify-content:center; border-radius:50%; width:30px; height:30px; transition:0.2s; margin: 0 auto;}
.btn-abort-row:hover { background: #ffebee; }
.modal { display:none; position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.6); z-index:999; align-items:center; justify-content:center; }
.modal-card { background:#fff; padding:20px; border-radius:18px; width:90%; max-width:380px; box-shadow:0 10px 30px rgba(0,0,0,0.3); }
.modal-card h3 { margin:0 0 15px 0; color:var(--dk); font-weight:800; }
.modal-card input, .modal-card select { width:100%; padding:10px; margin-bottom:15px; border:2px solid #e9ecef; border-radius:10px; font-family:inherit; outline:none; }
.modal-card input:focus, .modal-card select:focus { border-color: var(--b); }
.modal-btns { display:flex; justify-content:space-between; gap:10px; }
</style>
</head>
<body>

<header>
    <div class="tt">
        <h1 id="ttl"></h1>
        <div class="clk" id="clk"></div>
    </div>
    <div class="btns">
        <button class="btn blue hid" id="back" onclick="home()">⬅️ <span>PRENSAS</span></button>
        <button class="btn red hid" id="btn-pdf" onclick="descargarPDF()">📥 <span>PDF</span></button>
        <button class="btn" onclick="document.fullscreenElement?document.exitFullscreen():document.documentElement.requestFullscreen()">⏱️ <span>PANTALLA</span></button>
        <label class="btn">📂 <span>CARGAR CSV</span><input type="file" accept=".csv" id="f"></label>
    </div>
</header>

<div id="home" class="card"><div class="grid" id="grid"></div></div>

<div id="det" class="card hid">
    <div class="sum" id="sum"></div>
    <div class="hd"><span>Sebango</span><span>Process name</span><span>Horas de alcance</span><span></span></div>
    <div id="list"></div>
</div>

<div class="foot" id="foot"></div>

<!-- MODAL DE ABORTO -->
<div class="modal" id="abortModal">
    <div class="modal-card">
        <h3>🛑 Registrar Aborto</h3>
        <p style="margin-top:0;">Sebango: <strong id="modalSebango" style="color:var(--C); font-size:1.1rem;"></strong></p>
        <label><strong>Piezas Producidas:</strong></label>
        <input type="number" id="abortQty" min="0" placeholder="Ej. 1500">
        <label><strong>Motivo de Aborto:</strong></label>
        <select id="abortReason">
            <option value="Troquel">Problema de Troquel</option>
            <option value="Mala Operación">Mala Operación</option>
            <option value="Blank Defectuoso">Blank Defectuoso</option>
            <option value="Falta de Material">Falta de Material</option>
        </select>
        <div class="modal-btns">
            <button class="btn" style="background:var(--mut);" onclick="cerrarModal()">Cancelar</button>
            <button class="btn red" onclick="confirmarAborto()">Registrar y Avanzar</button>
        </div>
    </div>
</div>

<script>
// ==========================================
// CONFIGURACIÓN DE CALENDARIOS Y VARIABLES
// ==========================================
const calendariosOctubre = {
    "LPS150":       { 11: "D", 12: "A", 13: "B", 14: "A", 15: "B", 16: "A" },
    "TR800 - TR2":  { 11: "C", 12: "B", 13: "A", 14: "B", 15: "C", 16: "D" },
    "TR1600 - TR3": { 11: "C", 12: "B", 13: "A", 14: "B", 15: "A", 16: "B" },
    "TR800 - TR4":  { 11: "A", 12: "B", 13: "A", 14: "B", 15: "C", 16: "D" },
    "PG300 - PG1":  { 11: "G", 12: "B", 13: "C", 14: "D", 15: "E", 16: "F" },
    "PG200 - PG2":  { 11: "C", 12: "D", 13: "A", 14: "B", 15: "C", 16: "D" },
    "PG300 - PG3":  { 11: "F", 12: "B", 13: "C", 14: "D", 15: "E", 16: "B" },
    "PG200 - PG4":  { 11: "B", 12: "C", 13: "D", 14: "E", 15: "B", 16: "C" },
    "TD200":        { 11: "E", 12: "B", 13: "A", 14: "B", 15: "A", 16: "B" }
};

let DATA=[], src="INVENTARIO_GENERAL.csv", sel=null, upd=new Date(), syncTS=Date.now();
let abortHistory=[], currentAbortItem=null;
const CSV_URL="INVENTARIO_GENERAL.csv", REFRESH_MS=60000;
const PR=["LPS150","TD200","TR1600 - TR1","TR800 - TR2","TR1600 - TR3","TR800 - TR4","PG300 - PG1","PG200 - PG2","PG300 - PG3","PG200 - PG4"];

/* Umbrales de Urgencia (Horas) */ 
const T={C:2.01, A:3.01, X:8.01};
const $=id=>document.getElementById(id), ORD=["C","A","X","K"];
const cl=h=>h<T.C?"C":h<T.A?"A":h<T.X?"X":"K";
const LB={C:"CRÍTICO",A:"ATENCIÓN",X:"ACCIÓN",K:"CALMA"}, EM={C:"🚨",A:"⚠️",X:"⚙️",K:"✅"};
const PTS=[[0,5],[4.1,44],[12.6,70],[48.6,90],[120,100]];

function pct(h){for(let i=1;i<PTS.length;i++){if(h<=PTS[i][0]){const[a,b]=[PTS[i-1],PTS[i]];return a[1]+(b[1]-a[1])*(h-a[0])/(b[0]-a[0])}}return 100}
function calcularPatron(prensa) {
    if (!prensa || !calendariosOctubre[prensa]) return "--";
    const now = new Date();
    const turno = (now.getHours() >= 6 && now.getHours() < 18) ? 1 : 2;
    return `${calendariosOctubre[prensa][now.getDate()] || "?"}${turno}`;
}

// ==========================================
// RENDERIZADO (DASHBOARD Y DETALLES)
// ==========================================
function home(){sel=null;render()}
function render(){
    const elapsed = (Date.now() - syncTS) / 3600000; // Horas transcurridas
    
    $("home").classList.toggle("hid",!!sel);
    $("det").classList.toggle("hid",!sel);
    $("back").classList.toggle("hid",!sel);
    $("btn-pdf").classList.toggle("hid",!sel);
    
    $("ttl").innerHTML = sel 
        ? `🏭 ${sel} <span style="color:var(--A); font-size:0.8em; margin-left:10px;">[Patrón: ${calcularPatron(sel)}]</span>` 
        : "🚨 Andon de Estampado – Selecciona prensa";
        
    if(!sel){
        $("grid").innerHTML=PR.map(p=>{
            const r = DATA.filter(x=>x.p===p && !x.ab);
            const m = r.length ? Math.min(...r.map(x=>Math.max(0, x.h - elapsed))) : 999;
            const c = cl(m);
            const n = r.filter(x=>cl(Math.max(0, x.h - elapsed))==="C").length;
            return `<button class="press ${c}" onclick="sel='${p}';render()"><b>${p}</b><small>${r.length} sebangos</small><em>${EM[c]}</em><strong>${n?n+" CRÍTICO"+(n>1?"S":""):LB[c]}</strong></button>`;
        }).join("");
    }else{
        const r = DATA.filter(x=>x.p===sel && !x.ab).map(x=>({...x, curH: Math.max(0, x.h - elapsed)})).sort((a,b)=>a.curH-b.curH);
        const k = {C:0, A:0, X:0, K:0}; 
        r.forEach(x=>k[cl(x.curH)]++);
        
        $("sum").innerHTML=ORD.map(c=>`<div class="${c}"><span>${k[c]}</span><small>${EM[c]} ${LB[c]}</small></div>`).join("");
        
        $("list").innerHTML=r.map(x=>{
            const c = cl(x.curH);
            return `<div class="row ${c}"><span class="s">${x.s}</span><span class="p">${x.p}</span><span class="h">${x.curH.toFixed(1)}h<small>${EM[c]} ${LB[c]}</small></span><button class="btn-abort-row" onclick="abrirModal('${x.s}')" title="Abortar Producción">🛑</button><div class="bar"><i class="bg${c}" style="width:${pct(x.curH)}%"></i></div></div>`;
        }).join("")||"<p style='text-align:center; grid-column: 1 / -1; font-weight:bold; color:var(--mut)'>Sin sebangos activos</p>";
        fit();
    }
    $("foot").textContent="Último Censo: "+new Date(syncTS).toLocaleString("es-MX")+" · "+src;
}

function fit(){
    const L=$("list"),n=L.children.length;if(!n||!sel)return;
    const W=L.clientWidth,H=L.clientHeight,g=6,maxC=Math.max(1,Math.floor(W/300));
    let c=1;for(;c<maxC;c++){const r=Math.ceil(n/c);if((H-g*(r-1))/r>=60)break}
    const r=Math.ceil(n/c);let rh=(H-g*(r-1))/r;rh=Math.min(Math.max(rh,44),96);
    L.style.setProperty("--rh",rh+"px");L.style.gridTemplateRows=`repeat(${r},${rh}px)`;L.style.gridAutoColumns="minmax(0,1fr)"
}
addEventListener("resize",fit);
document.addEventListener("fullscreenchange",()=>setTimeout(fit,150));
const tick=()=>$("clk").textContent=new Date().toLocaleString("es-MX");setInterval(tick,1000);tick();setInterval(render,30000);

// ==========================================
// PARSER CSV
// ==========================================
function parse(t){const R=[];let r=[],c="",q=false;for(let i=0;i<t.length;i++){const ch=t[i];if(q){if(ch=='"'){if(t[i+1]=='"'){c+='"';i++}else q=false}else c+=ch}else if(ch=='"')q=true;else if(ch==","){r.push(c);c=""}else if(ch=="\n"){r.push(c);R.push(r);r=[];c=""}else if(ch!="\r")c+=ch}r.push(c);R.push(r);return R}
function toData(buf){
    const rows=parse(new TextDecoder("latin1").decode(buf));const hi=rows.findIndex(r=>r.some(v=>v.trim().toUpperCase()==="SEBANGO"));if(hi<0)throw 0;
    const H=rows[hi].map(v=>v.trim().toUpperCase()),iS=H.indexOf("SEBANGO"),iP=H.indexOf("PROCESS NAME"),iH=H.indexOf("SCOPE HOURS");
    // Se estructura en Objetos: {s: sebango, p: process, h: hours, ab: aborted}
    const d=rows.slice(hi+1).filter(r=>r[iS]&&r[iS].trim()&&r[iP]&&r[iP].trim()).map(r=>({s:r[iS].trim(), p:r[iP].trim(), h:parseFloat((r[iH]||"0").replace(/,/g,""))||0, ab:false}));
    if(!d.length)throw 0;return d;
}

// ==========================================
// ABORTO Y PDF
// ==========================================
function abrirModal(sebango){ currentAbortItem = sebango; $("modalSebango").innerText = sebango; $("abortQty").value = ''; $("abortModal").style.display = 'flex'; }
function cerrarModal(){ $("abortModal").style.display = 'none'; currentAbortItem = null; }
function confirmarAborto(){
    const qty = $("abortQty").value, reason = $("abortReason").value;
    if(!qty) return alert("Ingrese la cantidad producida.");
    
    abortHistory.push({ sebango: currentAbortItem, piezas: qty, motivo: reason, fecha: new Date().toLocaleString('es-MX') });
    const idx = DATA.findIndex(x => x.s === currentAbortItem);
    if(idx > -1) DATA[idx].ab = true;
    
    cerrarModal(); render();
}
function descargarPDF(){
    if(abortHistory.length === 0) return alert("No hay registros de abortos en esta sesión.");
    const doc = new window.jspdf.jsPDF();
    doc.text(`Reporte de Abortos - ${sel}`, 14, 20);
    doc.autoTable({
        head: [["Sebango", "Piezas", "Motivo", "Fecha/Hora"]],
        body: abortHistory.filter(r => DATA.find(x=>x.s===r.sebango)?.p === sel).map(r => [r.sebango, r.piezas, r.motivo, r.fecha]),
        startY: 30, theme: 'grid'
    });
    doc.save(`Abortos_${sel.replace(/\s/g,'')}_${new Date().getTime()}.pdf`);
}

// ==========================================
// CARGA LOCAL Y REMOTA
// ==========================================
$("f").onchange=async e=>{
    const fl=e.target.files[0];if(!fl)return;
    try{DATA=toData(await fl.arrayBuffer()); src=fl.name; upd=new Date(); syncTS=Date.now(); render();}
    catch(x){alert("No se pudo leer el CSV. Revise que contenga las columnas: SEBANGO, Process Name, SCOPE HOURS.");}
};

async function loadRemote(){
    try{
        const r=await fetch(CSV_URL+"?t="+Date.now(),{cache:"no-store"});if(!r.ok)throw 0;
        DATA=toData(await r.arrayBuffer());const lm=r.headers.get("Last-Modified");upd=lm?new Date(lm):new Date();src=CSV_URL;syncTS=Date.now();render();
    }catch(e){
        if(!DATA.length)$("foot").textContent="⚠️ Usa CARGAR CSV para inyectar la base de datos de inventario localmente.";
    }
}
loadRemote();
setInterval(loadRemote, REFRESH_MS);
render();
</script>
</body>
</html>
