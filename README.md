<!DOCTYPE html>
<html lang="ku">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>zand cafe</title>
<style>
:root{--bg:#020617;--card:#0f172a;--text:#e5e7eb;--income:#22c55e;--expense:#ef4444;--accent:#38bdf8}
*{box-sizing:border-box;font-family:Segoe UI,sans-serif}
body{margin:0;background:var(--bg);color:var(--text);padding:20px}
.container{max-width:1200px;margin:auto}
h1{text-align:center}
.card{background:var(--card);padding:15px;border-radius:12px;margin-bottom:15px}
input,select,button{width:100%;padding:10px;margin-top:8px;border-radius:8px;border:none}
button{background:var(--accent);font-weight:bold;cursor:pointer}
.summary{display:flex;gap:10px}
.sum{flex:1;padding:15px;border-radius:10px;text-align:center}
.income{background:#052e16;color:var(--income)}
.expense{background:#3f0d0d;color:var(--expense)}
.profit{background:#082f49;color:var(--accent)}
table{width:100%;border-collapse:collapse}
th,td{padding:8px;border-bottom:1px solid #1e293b;text-align:center}
.small{font-size:12px;opacity:.7}
.clock{text-align:center;font-size:18px;margin-bottom:10px}
</style>
</head>
<body>
<div class="container">
<h1>☕ zand cafe</h1>
<div class="clock" id="clock">--:--:--</div>

<div class="card">
<h2>🔐 هەژماری بەکارهێنەر</h2>
<select id="roleSelect">
  <option value="owner">Owner</option>
  <option value="staff">Staff</option>
</select>
<input type="password" id="passInput" placeholder="پاسۆرد">
<button onclick="login()">چوونەژوورەوە</button>
<p class="small">Owner دەتوانێت تۆمار و راپۆرت بینێت، Staff تەنها زیادکردن دەتوانێت بکات</p>
</div>

<div class="card" id="recordSection" style="display:none">
<h2>📥 / 📤 تۆمار</h2>
<select id="kind">
  <option value="income">داهات</option>
  <option value="expense">خەرجی</option>
</select>
<input type="text" id="note" placeholder="تێبینی (چی کڕراوە؟)">
<input type="number" id="amount" placeholder="بڕی پارە">
<button onclick="addRecord()">زیادکردن</button>
</div>

<div class="summary" id="summarySection" style="display:none">
<div class="sum income">📈 کۆی داهات<br><span id="ti">0</span></div>
<div class="sum expense">📉 کۆی خەرجی<br><span id="te">0</span></div>
<div class="sum profit">💰 قازانج<br><span id="profit">0</span></div>
</div>

<div class="card" id="reportSection" style="display:none">
<h2>📅 ڕاپۆرتی مانگانە و ساڵانە</h2>
<label>مانگ دیاری بکە:</label>
<select id="monthSelect" onchange="renderMonthReport()">
  <option value="0">1</option>
  <option value="1">2</option>
  <option value="2">3</option>
  <option value="3">4</option>
  <option value="4">5</option>
  <option value="5">6</option>
  <option value="6">7</option>
  <option value="7">8</option>
  <option value="8">9</option>
  <option value="9">10</option>
  <option value="10">11</option>
  <option value="11">12</option>
</select>
<label>ساڵ دیاری بکە:</label>
<select id="yearSelect" onchange="renderMonthReport()">
    <option value="2026">2026</option>
  <option value="2025">2027</option>
  <option value="2024">2028</option>
  <option value="2023">2029</option>
  

</select>
<div class="summary" style="margin-top:10px">
<div class="sum income">📈 کۆی داهات: <span id="repIncome">0</span></div>
<div class="sum expense">📉 کۆی خەرجی: <span id="repExpense">0</span></div>
<div class="sum profit">💰 قازانج: <span id="repProfit">0</span></div>
</div>
</div>

<div class="card" id="logSection" style="display:none">
<h2>📋 تۆمارەکان</h2>
<table>
<thead><tr><th>بەروار</th><th>سەعات</th><th>جۆر</th><th>تێبینی</th><th>بڕ</th><th>کردار</th></tr></thead>
<tbody id="log"></tbody>
</table>
</div>

<script>
const PASSWORDS = {owner:'zand', staff:'staff'};
let records = JSON.parse(localStorage.getItem('records')||'[]');
let role = '';
let loggedIn = false;

function login(){
  const selectedRole = document.getElementById('roleSelect').value;
  const pass = document.getElementById('passInput').value;
  if(PASSWORDS[selectedRole] && pass === PASSWORDS[selectedRole]){
    role = selectedRole;
    loggedIn = true;
    document.getElementById('recordSection').style.display = 'block';
    if(role === 'owner'){
      document.getElementById('summarySection').style.display='block';
      document.getElementById('reportSection').style.display='block';
      document.getElementById('logSection').style.display='block';
      renderMonthReport();
    }
  } else {
    alert('پاسۆرد هەڵەیە');
  }
}

function addRecord(){
  if(!loggedIn){ alert('سەرەتا چوونەژوورەوە بکە'); return; }
  if(role==='owner'){ if(!confirm('دڵنیایت کە داهات یان خەرجی زیاد دەکەیت؟')) return; }
  const amount = +document.getElementById('amount').value;
  if(!amount) return;
  const now = new Date();
  records.push({
    kind: document.getElementById('kind').value==='income'?'داهات':'خەرجی',
    note: document.getElementById('note').value,
    amount,
    date: now.toISOString(),
    time: now.toLocaleTimeString()
  });
  localStorage.setItem('records',JSON.stringify(records));
  document.getElementById('amount').value='';
  document.getElementById('note').value='';
  if(role==='owner') renderMonthReport();
}

function delRec(i){
  if(role!=='owner') return;
  records.splice(i,1);
  localStorage.setItem('records',JSON.stringify(records));
  renderMonthReport();
}

function renderMonthReport(){
  if(role!=='owner') return;
  const selectedMonth = parseInt(document.getElementById('monthSelect').value);
  const selectedYear = parseInt(document.getElementById('yearSelect').value);
  let ti=0, te=0;
  const log=document.getElementById('log');
  log.innerHTML='';
  records.forEach((r,i)=>{
    const d = new Date(r.date);
    if(d.getMonth() !== selectedMonth || d.getFullYear() !== selectedYear) return;
    log.innerHTML += `<tr>
      <td>${d.toLocaleDateString()}</td>
      <td>${r.time}</td>
      <td>${r.kind}</td>
      <td>${r.note||'-'}</td>
      <td>${r.amount}</td>
      <td><button onclick="delRec(${i})">سڕینەوە</button></td>
    </tr>`;
    if(r.kind==='داهات') ti+=r.amount; else te+=r.amount;
  });
  document.getElementById('ti').innerText=ti;
  document.getElementById('te').innerText=te;
  document.getElementById('profit').innerText=ti-te;
  document.getElementById('repIncome').innerText=ti;
  document.getElementById('repExpense').innerText=te;
  document.getElementById('repProfit').innerText=ti-te;
}

function updateClock(){ document.getElementById('clock').innerText = new Date().toLocaleTimeString(); }
setInterval(updateClock,1000); updateClock();
</script>
</body>
</html>
