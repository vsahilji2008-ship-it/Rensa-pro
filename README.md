# Rensa-pro
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Rensa Pro - Study Tracker</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap');
  body{margin:0;font-family:'Poppins',sans-serif;background:#0d0d16;color:#fff;}
  header{background:linear-gradient(135deg,#4f46e5,#9333ea);padding:35px;text-align:center;box-shadow:0 5px 15px #0005;}
  header h1{margin:0;font-size:40px;letter-spacing:2px;}
  header p{opacity:0.85;font-size:16px;}
  .container{max-width:950px;margin:20px auto;padding:15px;}
  .card{background:#141423;margin-bottom:20px;padding:20px;border-radius:14px;box-shadow:0 0 18px #0004;border:1px solid #2d2d44;}
  h2{margin:0 0 10px 0;color:#8b5cf6;font-size:22px;}
  input,select,textarea{width:100%;padding:10px;margin-top:8px;border:none;border-radius:8px;background:#1d1d2d;color:white;font-size:15px;}
  button{margin-top:12px;padding:10px 18px;background:linear-gradient(135deg,#4f46e5,#9333ea);border:none;color:white;border-radius:8px;font-size:15px;cursor:pointer;}
  .progress{background:#1f1f33;height:20px;border-radius:10px;margin-top:10px;overflow:hidden;}
  .progress-bar{height:100%;background:linear-gradient(90deg,#4f46e5,#9333ea);width:0%;transition:0.5s;}
  .flex{display:flex;gap:10px;flex-wrap:wrap;align-items:center;}
  .topic{display:flex;justify-content:space-between;align-items:center;padding:8px;margin-bottom:8px;border-radius:8px;background:#1d1d2d;}
  .topic.done{background:#0f1e16;border-left:5px solid #10b981;}
  .pie-wrapper{width:120px;height:120px;position:relative;}
  .pie-wrapper canvas{position:absolute;top:0;left:0;}
  .pie-text{position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);font-size:18px;font-weight:600;color:white;text-align:center;}
  footer{margin-top:30px;padding:12px;text-align:center;background:#10101a;color:white;font-size:14px;opacity:0.85;}
</style>
</head>
<body>

<header>
  <h1>Rensa Pro</h1>
  <p>Track your study, syllabus & daily goals like a pro!</p>
</header>

<div class="container">

  <!-- LOGIN/SIGNUP -->
  <div id="loginCard" class="card">
    <h2>👤 Login / Signup</h2>
    <input type="text" id="username" placeholder="Enter your username">
    <button onclick="login()">Login / Signup</button>
  </div>

  <!-- MAIN APP -->
  <div id="app" style="display:none;">

    <!-- Daily Progress -->
    <div class="card">
      <h2>📘 Daily Study Progress</h2>
      <input type="number" id="dailyStudy" oninput="updateProgress('dailyStudy','dailyStudyBar')" placeholder="Enter % completed today">
      <div class="progress"><div id="dailyStudyBar" class="progress-bar"></div></div>
    </div>

    <!-- Subject Tracker -->
    <div class="card">
      <h2>📚 Subjects & Syllabus</h2>
      <div class="flex">
        <input type="text" id="newSubject" placeholder="Add new subject">
        <button onclick="addSubject()">Add</button>
      </div>
      <div id="subjectList" style="margin-top:15px"></div>
    </div>

    <!-- Pie Chart -->
    <div class="card">
      <h2>📊 Completion Overview</h2>
      <div class="flex" style="gap:25px;align-items:center;">
        <div class="pie-wrapper">
          <canvas id="pieChart" width="120" height="120"></canvas>
          <div class="pie-text" id="pieText">0%</div>
        </div>
        <div class="small" style="color:#aaa">Green = Completed Topics</div>
      </div>
    </div>

    <!-- Notes -->
    <div class="card">
      <h2>📝 Notes / Goals</h2>
      <textarea id="notes" rows="3" placeholder="Write your notes / goals..."></textarea>
      <button onclick="saveNotes()">Save Notes</button>
    </div>

    <!-- Export / Import -->
    <div class="card">
      <h2>💾 Export / Import Data</h2>
      <button onclick="exportData()">Export JSON</button>
      <input type="file" id="importFile" onchange="importData(event)">
    </div>

  </div>
</div>

<footer>
  © 2025 Rensa Pro - All Rights Reserved
</footer>

<script>
let state = {}, currentUser="";

function login(){
  const uname = document.getElementById("username").value.trim();
  if(!uname){alert("Enter username");return;}
  currentUser = uname;
  const data = JSON.parse(localStorage.getItem("rensaPro"))||{};
  if(!data[currentUser]) data[currentUser]={subjects:{},dailyStudy:0,notes:""};
  state = data[currentUser];
  localStorage.setItem("rensaPro",JSON.stringify(data));
  document.getElementById("loginCard").style.display="none";
  document.getElementById("app").style.display="block";
  renderSubjects(); updateProgress('dailyStudy','dailyStudyBar'); document.getElementById("notes").value=state.notes||"";
  updatePie();
}

function saveState(){ 
  const data = JSON.parse(localStorage.getItem("rensaPro"))||{};
  data[currentUser]=state;
  localStorage.setItem("rensaPro",JSON.stringify(data));
}

function updateProgress(inputId,barId){
  let val = document.getElementById(inputId).value;
  if(val>100) val=100;if(val<0) val=0;
  document.getElementById(barId).style.width=val+"%";
  if(inputId==='dailyStudy') state.dailyStudy=val;
  saveState();
  updatePie();
}

function addSubject(){
  const name = document.getElementById("newSubject").value.trim();
  if(!name){alert("Enter subject name"); return;}
  const id = "sub_"+Math.random().toString(36).substr(2,6);
  state.subjects[id]={name:name,topics:[]};
  document.getElementById("newSubject").value="";
  saveState(); renderSubjects(); updatePie();
}

function renderSubjects(){
  const list = document.getElementById("subjectList");
  list.innerHTML="";
  Object.keys(state.subjects).forEach(id=>{
    const sub = state.subjects[id];
    const div = document.createElement("div");
    div.className="card";
    let topicsHtml="";
    if(sub.topics.length==0) topicsHtml='<div style="color:#aaa;font-size:14px">No topics yet</div>';
    else sub.topics.forEach((t,i)=>{
      topicsHtml+=`<div class="topic ${t.done?'done':''}">
        ${t.title} 
        <button onclick="toggleTopic('${id}',${i})" style="background:none;border:none;color:#4f46e5;cursor:pointer">${t.done?'Undo':'Done'}</button>
      </div>`;
    });
    div.innerHTML=`<h3 style="margin-top:0">${sub.name}</h3>
      <input type="text" id="topic_${id}" placeholder="Add topic">
      <button onclick="addTopic('${id}')">Add Topic</button>
      ${topicsHtml}`;
    list.appendChild(div);
  });
}

function addTopic(subId){
  const input = document.getElementById("topic_"+subId);
  const val = input.value.trim();
  if(!val){alert("Enter topic name"); return;}
  state.subjects[subId].topics.push({title:val,done:false});
  input.value="";
  saveState(); renderSubjects(); updatePie();
}

function toggleTopic(subId,idx){
  const t = state.subjects[subId].topics[idx];
  t.done=!t.done;
  saveState(); renderSubjects(); updatePie();
}

function updatePie(){
  const ctx = document.getElementById("pieChart").getContext("2d");
  const allTopics = [], doneTopics = [];
  Object.values(state.subjects).forEach(sub=>{
    sub.topics.forEach(t=>{allTopics.push(t); if(t.done) doneTopics.push(t);});
  });
  const pct = allTopics.length===0?0:Math.round(doneTopics.length/allTopics.length*100);
  document.getElementById("pieText").innerText=pct+"%";
  // draw
  ctx.clearRect(0,0,120,120);
  ctx.beginPath();
  ctx.moveTo(60,60);
  ctx.arc(60,60,50,0,2*Math.PI);
  ctx.fillStyle="#1f1f33"; ctx.fill();
  ctx.beginPath();
  ctx.moveTo(60,60);
  ctx.arc(60,60,50,-0.5*Math.PI,(-0.5+2*pct/100)*Math.PI);
  ctx.fillStyle="#10b981"; ctx.fill();
}

function saveNotes(){
  state.notes=document.getElementById("notes").value;
  saveState(); alert("Notes saved!");
}

function exportData(){
  const blob = new Blob([JSON.stringify(state,null,2)],{type:"application/json"});
  const url=URL.createObjectURL(blob);
  const a=document.createElement("a"); a.href=url; a.download="rensaPro_"+currentUser+".json"; a.click();
  URL.revokeObjectURL(url);
}

function importData(e){
  const file=e.target.files[0];
  const reader=new FileReader();
  reader.onload=function(ev){
    try{
      const imported=JSON.parse(ev.target.result);
      state=imported; saveState(); renderSubjects(); document.getElementById("dailyStudy").value=state.dailyStudy||0;
      updateProgress('dailyStudy','dailyStudyBar'); document.getElementById("notes").value=state.notes||""; alert("Imported successfully!");
    }catch(err){alert("Invalid JSON file");}
  }
  reader.readAsText(file);
}
</script>

</body>
</html>
