<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>VIPZEXNET - چت هوش مصنوعی</title>
<style>
:root { --bg-dark:#0d1117; --text-light:#fff; --brand:#58a6ff; --success:#238636; --card-dark:rgba(33,38,45,0.9);}
body { margin:0; font-family:sans-serif; background:var(--bg-dark); color:var(--text-light); display:flex; flex-direction:column; height:100vh; position:relative;}
.background-text { position: fixed; top:50%; left:50%; transform:translate(-50%,-50%); font-size:60px; font-weight:bold; color:rgba(255,255,255,0.05); user-select:none; pointer-events:none; z-index:0;}
header { background:#161b22; padding:10px; display:flex; align-items:center; justify-content:space-between; border-bottom:1px solid #30363d; font-size:16px; z-index:1; position:relative; }
header h1 { margin:0; font-size:16px; color:var(--brand); flex:1; white-space:nowrap; overflow:hidden; text-overflow:ellipsis;}
#chatWindow { flex:1; overflow-y:auto; padding:10px; display:flex; flex-direction:column; gap:8px; z-index:1; position:relative; }
.message { max-width:85%; padding:10px 14px; border-radius:12px; line-height:1.4; white-space:pre-wrap; position:relative; animation:fadeIn 0.3s ease-in-out; word-break: break-word; font-size:14px; display:flex; align-items:flex-start; gap:5px; }
.user { background:var(--success); color:#fff; align-self:flex-end; border-bottom-right-radius:0; }
.bot { background:var(--card-dark); color:#fff; align-self:flex-start; border-bottom-left-radius:0; }
.msg-time { font-size:10px; opacity:0.6; text-align:left; margin-top:2px; }
.msg-user { font-weight:bold; font-size:12px; }
#inputArea { display:flex; padding:8px; border-top:1px solid #30363d; background:#161b22; gap:5px; align-items:center; z-index:1; }
#userInput { flex:1; padding:10px; border:none; border-radius:8px; background:var(--bg-dark); color:var(--text-light); font-size:15px; }
#sendBtn, #uploadBtn { padding:10px 12px; background:var(--brand); border:none; border-radius:8px; cursor:pointer; color:#fff; font-size:14px; }
#sendBtn:hover, #uploadBtn:hover { background:#1f6feb; }
#historyMenu { display:none; position:absolute; top:40px; right:0; background:#161b22; border:1px solid #30363d; border-radius:6px; min-width:180px; z-index:100; max-height:250px; overflow-y:auto; font-size:14px; }
#historyMenu button { width:100%; padding:6px; border:none; background:none; color:#fff; text-align:right; cursor:pointer; font-size:14px; }
#historyMenu button:hover { background:#30363d; }
#menuContainer { position:relative; z-index:1; }
#profileModal { display:none; position:fixed; top:50%; left:50%; transform:translate(-50%,-50%); background:#161b22; border:1px solid #30363d; border-radius:10px; padding:15px; z-index:200; width:90%; max-width:300px; color:#fff; }
#profileModal input { width:100%; margin-bottom:8px; padding:6px; background:#0d1117; color:#fff; border:1px solid #30363d; border-radius:5px; font-size:14px; }
#profileModal button { cursor:pointer; border:none; border-radius:5px; padding:5px 10px; font-size:14px; }
#saveProfile { background:var(--success); color:#fff; margin-left:5px; }
#closeProfile { background:#30363d; color:#fff; }
.typing { font-style:italic; opacity:0.7; }
.message img { max-width:120px; border-radius:50%; display:block; }
@keyframes fadeIn { from{opacity:0; transform:translateY(5px);} to{opacity:1; transform:translateY(0);} }
</style>
</head>
<body>

<div class="background-text">من VIPZEXNET هستم</div>

<header>
  <h1>🤖 VIPZEXNET</h1>
  <div id="menuContainer">
    <button id="menuBtn" style="background:none;border:none;color:var(--brand);font-size:18px;cursor:pointer;">⋮</button>
    <div id="historyMenu">
      <button id="newChat">🆕 چت جدید</button>
      <button id="profileBtn">👤 پروفایل</button>
      <div id="chatHistoryList"></div>
    </div>
  </div>
</header>

<div id="chatWindow"></div>

<div id="inputArea">
  <button id="uploadBtn">📎</button>
  <input type="text" id="userInput" placeholder="پیام خود را بنویسید..." onkeydown="if(event.key==='Enter'){sendMessage()}">
  <button id="sendBtn" onclick="sendMessage()">ارسال</button>
</div>

<div id="profileModal">
  <h3>پروفایل</h3>
  <label>نام:</label><input type="text" id="profileName">
  <label>ایمیل:</label><input type="email" id="profileEmail">
  <label>عکس پروفایل:</label><input type="file" id="profileAvatar" accept="image/*">
  <div style="text-align:right;">
    <button id="saveProfile">ذخیره</button>
    <button id="closeProfile">بستن</button>
  </div>
</div>

<input type="file" id="fileInput" style="display:none" accept="image/*,.txt">

<script>
let selectedModel="openai-large";
const chatWindow=document.getElementById("chatWindow");
const profileAvatar=document.getElementById("profileAvatar");

// بارگذاری تاریخچه
function loadChatHistory(){
  const history=JSON.parse(localStorage.getItem("chatHistory")||"[]");
  history.forEach(msg=>addMessage(msg.text,msg.sender,false,msg.time,msg.name,msg.img));
}

// ذخیره پیام
function saveMessage(text,sender,time,name=null,img=null){
  const history=JSON.parse(localStorage.getItem("chatHistory")||"[]");
  history.push({text,sender,time,name,img});
  localStorage.setItem("chatHistory",JSON.stringify(history));
}

// افزودن پیام با عکس پروفایل
function addMessage(text,sender,save=true,time=null,name=null,img=null,isTyping=false){
  const msgDiv=document.createElement("div");
  msgDiv.className="message "+sender;
  if(!time) time=new Date().toLocaleTimeString([], {hour:'2-digit',minute:'2-digit'});
  let profile=JSON.parse(localStorage.getItem("userProfile")||"{}");
  let userDisplay=name ? `<div class="msg-user">${name}</div>` : "";
  let avatarTag = sender==="user" && profile.avatar ? `<img src="${profile.avatar}" style="width:30px;height:30px;border-radius:50%;">` : "";
  let imgTag=img ? `<img src="${img}">` : "";
  msgDiv.innerHTML=`${avatarTag}${userDisplay}<span>${text}</span>${imgTag}<div class="msg-time">${time}</div>`;
  if(isTyping) msgDiv.classList.add("typing");
  chatWindow.appendChild(msgDiv);
  chatWindow.scrollTop=chatWindow.scrollHeight;
  if(save) saveMessage(text,sender,time,name,img);

  if(sender==="user"){
    msgDiv.addEventListener('dblclick',()=>{
      const newText=prompt("ویرایش پیام:",text);
      if(newText){
        msgDiv.querySelector("span").textContent=newText;
        updateMessageInHistory(text,newText);
      }
    });
  }

  return msgDiv;
}

// ویرایش پیام
function updateMessageInHistory(oldText,newText){
  const history=JSON.parse(localStorage.getItem("chatHistory")||"[]");
  const msg=history.find(m=>m.text===oldText && m.sender==="user");
  if(msg){ msg.text=newText; localStorage.setItem("chatHistory",JSON.stringify(history)); }
}

// ارسال پیام به API واقعی
async function sendMessage(){
  const input=document.getElementById("userInput");
  const text=input.value.trim();
  if(!text) return;
  const profile=JSON.parse(localStorage.getItem("userProfile")||"{}");
  addMessage(text,"user",true,null,profile.name);
  input.value="";

  const loadingMsg=addMessage("در حال دریافت پاسخ...","bot",false,true);

  try{
    const response = await fetch(`https://api2.api-code.ir/gpt-save-v2/?model=${selectedModel}&userid=mahdi&prompt=${encodeURIComponent(text)}`);
    const data = await response.text();
    loadingMsg.remove();
    addMessage(data,"bot");
  }catch(err){
    loadingMsg.remove();
    addMessage("❌ خطا در دریافت پاسخ","bot");
    console.error(err);
  }
}

// آپلود تصویر و فایل
const uploadBtn=document.getElementById("uploadBtn");
const fileInput=document.getElementById("fileInput");
uploadBtn.addEventListener("click",()=>fileInput.click());
fileInput.addEventListener("change",(e)=>{
  const file=e.target.files[0];
  if(!file) return;
  const reader=new FileReader();
  reader.onload=function(){
    const profile=JSON.parse(localStorage.getItem("userProfile")||"{}");
    if(file.type.startsWith("image/")){
      addMessage("", "user", true, null, profile.name, reader.result);
    } else if(file.type==="text/plain"){
      addMessage(reader.result, "user");
    }
  };
  reader.readAsDataURL(file);
});

// منوی سه نقطه
const menuBtn=document.getElementById("menuBtn");
const historyMenu=document.getElementById("historyMenu");
const chatHistoryList=document.getElementById("chatHistoryList");
menuBtn.addEventListener("click",()=>{
  historyMenu.style.display=historyMenu.style.display==="block"?"none":"block";
  renderHistoryList();
});
document.getElementById("newChat").addEventListener("click",()=>{
  if(confirm("آیا مطمئن هستید که چت جدید شروع شود؟")){
    localStorage.removeItem("chatHistory");
    chatWindow.innerHTML="";
    historyMenu.style.display="none";
  }
});
function renderHistoryList(){
  const history=JSON.parse(localStorage.getItem("chatHistory")||"[]");
  chatHistoryList.innerHTML="";
  history.forEach(msg=>{
    const btn=document.createElement("button");
    btn.textContent=(msg.sender==="user"?"👤 ":"🤖 ")+msg.text.slice(0,30);
    btn.addEventListener("click",()=>alert("پیام کامل:\n"+msg.text));
    chatHistoryList.appendChild(btn);
  });
}
document.addEventListener("click",(e)=>{
  if(!historyMenu.contains(e.target) && e.target!==menuBtn) historyMenu.style.display="none";
});

// پروفایل
const profileBtn=document.getElementById("profileBtn");
const profileModal=document.getElementById("profileModal");
const closeProfile=document.getElementById("closeProfile");
const saveProfile=document.getElementById("saveProfile");
const profileName=document.getElementById("profileName");
const profileEmail=document.getElementById("profileEmail");

profileBtn.addEventListener("click",()=>{
  profileModal.style.display="block";
  historyMenu.style.display="none";
  const profile=JSON.parse(localStorage.getItem("userProfile")||"{}");
  profileName.value=profile.name||"";
  profileEmail.value=profile.email||"";
});

closeProfile.addEventListener("click",()=>{profileModal.style.display="none";});

saveProfile.addEventListener("click",()=>{
  const profile={name:profileName.value.trim(),email:profileEmail.value.trim()};
  if(profileAvatar.files[0]){
    const reader = new FileReader();
    reader.onload = function(){
      profile.avatar = reader.result;
      localStorage.setItem("userProfile",JSON.stringify(profile));
      alert("پروفایل با عکس ذخیره شد ✅");
      profileModal.style.display="none";
    };
    reader.readAsDataURL(profileAvatar.files[0]);
  }else{
    const oldProfile=JSON.parse(localStorage.getItem("userProfile")||"{}");
    if(oldProfile.avatar) profile.avatar = oldProfile.avatar;
    localStorage.setItem("userProfile",JSON.stringify(profile));
    alert("پروفایل ذخیره شد ✅");
    profileModal.style.display="none";
  }
});

loadChatHistory();
</script>
</body>
</html>
