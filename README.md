[index.html](https://github.com/user-attachments/files/31929094/index.html)
<!doctype html>
<html lang="es">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Tirado Barber — Reserva tu cita</title>
<meta name="description" content="Reserva tu cita con Tirado Barber.">
<style>
*{box-sizing:border-box}body{margin:0;font-family:Inter,Arial,sans-serif;background:#0b0b0b;color:#fff}
a{color:inherit}.wrap{max-width:980px;margin:auto;padding:24px}
.hero{min-height:62vh;display:grid;place-items:center;text-align:center;padding:70px 20px;background:radial-gradient(circle at top,#252525,#0b0b0b 60%)}
.logo{font-size:52px;font-weight:900;letter-spacing:-2px}.sub{color:#bcbcbc;font-size:18px;margin:14px 0 30px}
.btn{display:inline-block;background:#fff;color:#000;border:0;border-radius:12px;padding:15px 22px;font-weight:800;text-decoration:none;cursor:pointer}
section{padding:55px 0}.grid{display:grid;grid-template-columns:repeat(3,1fr);gap:16px}.card{background:#151515;border:1px solid #292929;border-radius:16px;padding:22px}.price{font-size:22px;font-weight:800;margin-top:10px}
.booking{max-width:650px;margin:auto;background:#151515;border:1px solid #292929;border-radius:20px;padding:24px}
label{display:block;margin:16px 0 7px;font-weight:700}input,select{width:100%;padding:14px;border-radius:10px;border:1px solid #444;background:#0d0d0d;color:#fff;font-size:16px}
.note{font-size:13px;color:#aaa;margin-top:8px}.hidden{display:none}.success{padding:20px;background:#142417;border:1px solid #315b37;border-radius:14px;margin-top:18px}.error{padding:14px;background:#301616;border:1px solid #6b3030;border-radius:12px;margin-top:18px}
footer{border-top:1px solid #222;color:#888;text-align:center;padding:30px}.small{font-size:14px;color:#999}
@media(max-width:700px){.grid{grid-template-columns:1fr}.logo{font-size:40px}}
</style>
</head>
<body>
<header class="hero">
  <div>
    <div class="logo">TIRADO</div>
    <div class="sub">BARBER · Reserva tu cita online</div>
    <a class="btn" href="#reservar">RESERVAR CITA</a>
  </div>
</header>

<main class="wrap">
<section>
  <h2>Servicios</h2>
  <div class="grid">
    <div class="card"><h3>Degradado normal</h3><p>Degradado clásico.</p><div class="price">6 €</div></div>
    <div class="card"><h3>Tinte</h3><p>Corte + servicio de tinte.</p><div class="price">45 €</div></div>
    <div class="card"><h3>Otros</h3><p>Si quieres añadir más servicios, se pueden incorporar.</p></div>
  </div>
</section>

<section id="reservar">
  <div class="booking">
    <h2>Reserva tu cita</h2>
    <p class="small">Las citas se confirman automáticamente según la disponibilidad.</p>
    <form id="form">
      <label for="service">Servicio</label>
      <select id="service" required>
        <option value="">Selecciona...</option>
        <option value="Degradado normal">Degradado normal</option>
        <option value="Tinte">Tinte</option>
      </select>

      <label for="date">Día</label>
      <input id="date" type="date" required>

      <label for="time">Hora</label>
      <select id="time" required disabled><option>Selecciona primero el día</option></select>
      <div class="note">Lunes a viernes: 10:00–13:00 y 16:00–19:00 · Fines de semana: 10:00–13:00.</div>

      <label for="name">Nombre</label>
      <input id="name" maxlength="60" required placeholder="Tu nombre">

      <label for="phone">Teléfono</label>
      <input id="phone" type="tel" maxlength="20" required placeholder="Tu teléfono">

      <button class="btn" style="width:100%;margin-top:22px" type="submit">CONFIRMAR CITA</button>
    </form>
    <div id="msg"></div>
  </div>
</section>
</main>
<footer>
  <div>Tirado Barber · <a href="https://instagram.com/tirado_barber" target="_blank">@tirado_barber</a></div>
  <div class="small">La dirección se mostrará únicamente después de confirmar una cita.</div>
</footer>

<script>
/*
  CONFIGURACIÓN:
  Pega aquí la URL de tu Google Apps Script cuando lo publiques.
*/
const API_URL = "https://script.google.com/macros/s/AKfycbxRJwkPV4PcIQQCXnKKH2QIubgehiImYYIs_HEn3A2OPpg-NmC87FyNegUpm2N23go/exec";

const dateInput=document.getElementById("date");
const timeSelect=document.getElementById("time");
const form=document.getElementById("form");
const msg=document.getElementById("msg");
const today=new Date(); today.setHours(0,0,0,0);
dateInput.min=today.toISOString().slice(0,10);

dateInput.addEventListener("change", async ()=>{
  if(dateInput.value) await loadSlots(dateInput.value);
});

async function loadSlots(date){
  timeSelect.disabled=true;
  timeSelect.innerHTML="<option>Cargando horas...</option>";
  try{
    const r=await fetch(API_URL+"?action=slots&date="+encodeURIComponent(date));
    const data=await r.json();
    timeSelect.innerHTML="";
    if(!data.slots?.length){
      timeSelect.innerHTML="<option>No hay horas disponibles</option>";
      return;
    }
    timeSelect.innerHTML='<option value="">Selecciona una hora...</option>'+
      data.slots.map(x=>`<option value="${x}">${x}</option>`).join("");
    timeSelect.disabled=false;
  }catch(e){
    timeSelect.innerHTML="<option>Error al cargar horas</option>";
  }
}

form.addEventListener("submit",async e=>{
  e.preventDefault();
  msg.className=""; msg.textContent="Confirmando...";
  try{
    const body=new URLSearchParams({
      action:"book",
      service:document.getElementById("service").value,
      date:dateInput.value,
      time:timeSelect.value,
      name:document.getElementById("name").value,
      phone:document.getElementById("phone").value,
      duration:document.getElementById("service").value==="Tinte" ? "120" : "45"
    });
    const r=await fetch(API_URL,{method:"POST",body});
    const data=await r.json();
    if(!data.ok) throw new Error(data.error||"No se pudo reservar");
    form.classList.add("hidden");
    msg.className="success";
    msg.innerHTML="<strong>¡Cita confirmada!</strong><br><br>"+
      `${data.service}<br>${data.date} a las ${data.time}<br><br>`+
      `<strong>Dirección:</strong><br>${escapeHtml(data.address)}<br><br>`+
      `<span class="small">Guarda estos datos. La cita ya está confirmada.</span>`;
  }catch(err){
    msg.className="error";
    msg.textContent=err.message;
    if(dateInput.value) loadSlots(dateInput.value);
  }
});
function escapeHtml(s){return String(s).replace(/[&<>"']/g,m=>({"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#039;"}[m]))}
</script>
</body>
</html>
