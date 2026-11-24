<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8"><meta name="viewport" content="width=device-width,initial-scale=1">
<title>أدمن توكتوك Go - سوهاج</title>
<script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-firestore-compat.js"></script>
<style>*{box-sizing:border-box;font-family:system-ui}body{margin:0;background:#F8FAFC}.wrap{max-width:520px;margin:auto;padding:12px}.card{background:#fff;border-radius:16px;padding:14px;margin-bottom:12px;box-shadow:0 2px 10px #0001}input{width:100%;padding:12px;margin:6px 0;border:1px solid #ddd;border-radius:10px}button{width:100%;padding:13px;border:0;border-radius:12px;font-weight:800;font-size:16px;cursor:pointer}.btn-y{background:#FFC107}.btn-b{background:#111;color:#fff}.btn-g{background:#16a34a;color:#fff}.btn-call{background:#0ea5e9;color:#fff}.btn-red{background:#dc2626;color:#fff}.btn-orange{background:#f97316;color:#fff}.driver{border:1px solid #eee;border-radius:14px;padding:10px;margin-top:10px;display:flex;gap:10px;align-items:center}.driver img{width:58px;height:58px;border-radius:50%;object-fit:cover}.small{font-size:13px;color:#666}.hidden{display:none}.flex{display:flex;gap:8px}.flex button{flex:1}.stat{display:flex;gap:8px;flex-wrap:wrap}.stat div{flex:1;min-width:100px;background:#F8FAFC;border:1px solid #eee;border-radius:12px;padding:10px;text-align:center}.stat b{font-size:20px;display:block}</style>
</head>
<body>
<div class="wrap">
<div id="lockPage" class="card"><h3>🔐 أدمن توكتوك Go</h3><p class="small">الصفحة دي ليك انت بس.. محدش يعرفها</p><input id="adminPass" type="password" placeholder="كلمة سر الأدمن"><button class="btn-b" onclick="unlockAdmin()">دخول الأدمن ✅</button></div>
<div id="adminPage" class="card hidden">
<h3>🔐 لوحة الأدمن - سوهاج</h3>
<div class="stat"><div><b id="stApproved">0</b><span class="small">سواق مقبول ✅</span></div><div><b id="stPending">0</b><span class="small">مستني ⏳</span></div><div><b id="stRiders">0</b><span class="small">راكب 🙋</span></div><div><b id="stReq">0</b><span class="small">طلب 📩</span></div></div>
<h3>📢 تنبيه لكل السواقين</h3><input id="adminMsg" placeholder="اكتب التنبيه هنا.."><button class="btn-orange" onclick="sendAdminNotice()">ابعت التنبيه 📢</button><button class="btn-red" onclick="clearAdminNotice()" style="margin-top:6px">امسح التنبيه</button>
<div class="flex" style="margin-top:10px"><button class="btn-b" onclick="showAdminTab('drivers')">السواقين</button><button class="btn-y" onclick="showAdminTab('riders')">الركاب</button><button class="btn-g" onclick="showAdminTab('trips')">الرحلات</button></div>
<div id="admin-drivers"><h3>مستنيين الموافقة ⏳</h3><div id="pendingBox"></div><h3>مقبولين ✅</h3><div id="approvedBox"></div><h3>مرفوضين / محظورين ❌</h3><div id="rejectedBox"></div></div>
<div id="admin-riders" class="hidden"><h3>كل الركاب 🙋</h3><div id="ridersBox"></div></div>
<div id="admin-trips" class="hidden"><h3>كل الطلبات 🚀</h3><button class="btn-red" onclick="deleteAllRequests()">مسح كل الطلبات 🗑️</button><div id="tripsBox"></div></div>
</div>
</div>
<script>
const ADMIN_PASS="toktok123";
const firebaseConfig={apiKey:"AIzaSyD7f6PTn5vEsEddCtmTdj3EDVGAnRjm-_A",authDomain:"toktokgo-65af3.firebaseapp.com",projectId:"toktokgo-65af3",storageBucket:"toktokgo-65af3.firebasestorage.app",messagingSenderId:"586951920995",appId:"1:586951920995:web:6995f3d5b9fded3e16d756"};
firebase.initializeApp(firebaseConfig);const db=firebase.firestore();
function unlockAdmin(){if(document.getElementById('adminPass').value!==ADMIN_PASS)return alert('كلمة السر غلط ❌');document.getElementById('lockPage').classList.add('hidden');document.getElementById('adminPage').classList.remove('hidden');loadAdmin();}
function showAdminTab(t){['drivers','riders','trips'].forEach(k=>document.getElementById('admin-'+k).classList.toggle('hidden',t!==k));}
async function sendAdminNotice(){let txt=document.getElementById('adminMsg').value.trim();if(!txt)return alert('اكتب التنبيه');await db.collection('settings').doc('notice').set({text:txt,time:Date.now()});alert('اتبع التنبيه 📢');}
async function clearAdminNotice(){await db.collection('settings').doc('notice').delete();alert('اتمسح');}
function loadAdmin(){
db.collection('drivers').onSnapshot(snap=>{let pen='',app='',rej='',cA=0,cP=0;snap.forEach(doc=>{let d=doc.data();let h=`<div class="driver"><img src="${d.img||''}"><div style="flex:1"><b>${d.name}</b><br><span class="small">${d.phone} • بطاقة: ${d.national||''}</span><br><a href="${d.cardImg||''}" target="_blank">شوف البطاقة 📷</a></div></div>`;if(d.status==='pending'){cP++;pen+=h+`<div class="flex"><button class="btn-g" onclick="setDriverStatus('${doc.id}','approved')">قبول ✅</button><button class="btn-b" onclick="setDriverStatus('${doc.id}','rejected')">رفض ❌</button></div>`;}else if(d.status==='approved'){cA++;app+=h+`<div class="flex"><button class="btn-red" onclick="setDriverStatus('${doc.id}','rejected')">حظر 🚫</button><a href="tel:+20${(d.phone||'').replace(/^0/,'')}" style="flex:1"><button class="btn-call">اتصل 📞</button></a></div>`;}else{rej+=h+`<div class="flex"><button class="btn-g" onclick="setDriverStatus('${doc.id}','approved')">فك حظر ✅</button><button class="btn-red" onclick="deleteDriver('${doc.id}')">مسح نهائي 🗑️</button></div>`;}});document.getElementById('pendingBox').innerHTML=pen||'مفيش';document.getElementById('approvedBox').innerHTML=app||'مفيش';document.getElementById('rejectedBox').innerHTML=rej||'مفيش';document.getElementById('stApproved').innerText=cA;document.getElementById('stPending').innerText=cP;});
db.collection('riders').onSnapshot(snap=>{document.getElementById('stRiders').innerText=snap.size;let h='';snap.forEach(doc=>{let r=doc.data();h+=`<div class="driver"><div style="flex:1"><b>${r.name}</b><br><span class="small">${r.phone}</span></div><button class="btn-red" style="width:80px" onclick="deleteRider('${doc.id}')">مسح</button></div>`;});document.getElementById('ridersBox').innerHTML=h||'مفيش';});
db.collection('requests').orderBy('created','desc').limit(50).onSnapshot(snap=>{document.getElementById('stReq').innerText=snap.size;let h='';snap.forEach(doc=>{let r=doc.data();h+=`<div class="driver"><div style="flex:1"><b>${r.name} - ${r.phone||''}</b><br><span class="small">من: ${r.from} ➜ إلى: ${r.to}<br>${r.price} جنيه • ${r.status} • الكابتن: ${r.driverName||'لسه'}</span></div><button class="btn-red" style="width:80px" onclick="deleteRequest('${doc.id}')">مسح</button></div>`;});document.getElementById('tripsBox').innerHTML=h||'مفيش';});
}
async function setDriverStatus(id,st){await db.collection('drivers').doc(id).update({status:st});alert(st==='approved'?'تم القبول ✅':'تم الحظر 🚫');}
async function deleteDriver(id){if(!confirm('متأكد؟'))return;await db.collection('drivers').doc(id).delete();}
async function deleteRider(id){if(!confirm('متأكد؟'))return;await db.collection('riders').doc(id).delete();}
async function deleteRequest(id){if(!confirm('متأكد؟'))return;await db.collection('requests').doc(id).delete();}
async function deleteAllRequests(){if(!confirm('متأكد تمسح الكل؟'))return;let s=await db.collection('requests').get();for(let d of s.docs) await db.collection('requests').doc(d.id).delete();alert('اتمسح الكل');}
</script>
</body>
</html>
