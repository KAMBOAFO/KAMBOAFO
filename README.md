<!DOCTYPE html>
<html lang="en"><head><meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>School Manager</title>
<style>
:root{box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px);--bg:#f6f5f0;--card:#fff;--tx:#1d2420;--mu:#68716b;--ac:#1f6f4a;--ac2:#e3f1ea;--bd:#dedbd0;--bad:#b3402a}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#141815;--card:#1d231f;--tx:#e8ece9;--mu:#98a39b;--ac:#4fb883;--ac2:#23352b;--bd:#2f3832;--bad:#e8826c}}
:root[data-theme="dark"]{--bg:#141815;--card:#1d231f;--tx:#e8ece9;--mu:#98a39b;--ac:#4fb883;--ac2:#23352b;--bd:#2f3832;--bad:#e8826c}
html{scroll-padding-top:env(safe-area-inset-top,0px)}*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--tx);font:15px/1.45 system-ui,-apple-system,Segoe UI,Roboto,sans-serif}
header{padding:14px 16px 4px;max-width:1050px;margin:auto}h1{margin:0;font-size:20px}header p{margin:2px 0 0;color:var(--mu);font-size:13px}
nav{display:flex;gap:6px;overflow-x:auto;padding:8px 16px;max-width:1050px;margin:auto;position:sticky;top:env(safe-area-inset-top,0px);background:var(--bg);z-index:5}
nav button{border:1px solid var(--bd);background:var(--card);color:var(--tx);padding:7px 13px;border-radius:20px;font:inherit;white-space:nowrap;cursor:pointer}
nav button.on{background:var(--ac);border-color:var(--ac);color:#fff}
main{max-width:1050px;margin:auto;padding:6px 16px 40px}
.card{background:var(--card);border:1px solid var(--bd);border-radius:12px;padding:14px;margin-bottom:14px}.card h2{margin:0 0 10px;font-size:16px}
.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(150px,1fr));gap:10px;margin-bottom:14px}
.stat{background:var(--card);border:1px solid var(--bd);border-radius:12px;padding:11px}.stat span{display:block;color:var(--mu);font-size:12px}.stat b{font-size:17px}
.row{display:flex;flex-wrap:wrap;gap:8px;align-items:flex-end;margin-bottom:10px}
label{display:flex;flex-direction:column;font-size:11px;color:var(--mu);gap:2px}
input,select{font:inherit;font-size:14px;padding:7px 9px;border:1px solid var(--bd);border-radius:8px;background:var(--bg);color:var(--tx);min-width:0;max-width:190px}
.btn{background:var(--ac);color:#fff;border:0;padding:8px 14px;border-radius:8px;font:inherit;cursor:pointer}.btn.s{padding:3px 9px;font-size:12px}.btn.g{background:transparent;color:var(--bad);border:1px solid var(--bd)}
.tw{overflow-x:auto}table{width:100%;border-collapse:collapse;font-size:14px}th,td{text-align:left;padding:7px 6px;border-bottom:1px solid var(--bd);white-space:nowrap}th{color:var(--mu);font-weight:500;font-size:12px}
.bad{color:var(--bad);font-weight:600}.tag{background:var(--ac2);color:var(--ac);padding:2px 8px;border-radius:10px;font-size:12px}.mu{color:var(--mu);font-size:13px}.fi{width:88px}
</style></head><body>
<header><h1>🏫 School Manager</h1><p>Permanent pupil IDs · yearly records · fees · salaries · taxes · projects · GH₵</p></header>
<nav id="nav"></nav><main id="app"></main>
<script>
const CL=['Creche','Nursery 1','Nursery 2','KG 1','KG 2','Class 1','Class 2','Class 3','Class 4','Class 5','Class 6','JHS 1','JHS 2','JHS 3'];
const YRS=[];for(let y=2024;y<2037;y++)YRS.push(y+'/'+String(y+1).slice(2));
const CATS=['Electricity','Water','Internet','Rent','Maintenance','Stationery','Food supplies','Transport','Other'];
const STY=['SSNIT','PAYE','Withholding tax','Income tax','VAT/NHIL','Property rate','Business operating permit','Fire certificate','GES / accreditation','Insurance','Other'];
const SUBJ=['English','Mathematics','Integrated Science','Social Studies','RME','Ghanaian Language','Computing (ICT)','Creative Arts','French','Career Tech'];
const KEY='school-v2';let S;try{S=JSON.parse(localStorage.getItem(KEY))}catch(e){}
if(!S)S={year:'2026/27',term:1,nid:1,sid:1,ssE:5.5,ssR:13,open:0,rep:{},fees:{},pupils:[],enrol:[],pay:[],exp:[],staff:[],prl:[],stat:[],proj:[],led:[],att:[],res:[]};
const save=(c=1)=>{if(c)S.changed=Date.now();try{localStorage.setItem(KEY,JSON.stringify(S))}catch(e){}};
const $=i=>document.getElementById(i),sum=a=>a.reduce((x,y)=>x+(+y||0),0),today=()=>new Date().toISOString().slice(0,10);
const fmt=n=>'GH₵ '+(+n||0).toLocaleString('en-GH',{minimumFractionDigits:2,maximumFractionDigits:2});
const esc=s=>String(s==null?'':s).replace(/[&<>"]/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;'}[c]));
const yo=d=>{const[y,m]=d.split('-').map(Number);return m>=9?y+'/'+String(y+1).slice(2):(y-1)+'/'+String(y).slice(2)};
const shift=(y,k)=>{const a=+y.slice(0,4)+k;return a+'/'+String(a+1).slice(2)};
function fees(y){if(!S.fees[y]){const p=S.fees[shift(y,-1)];S.fees[y]=p?JSON.parse(JSON.stringify(p)):Object.fromEntries(CL.map((c,i)=>[c,{tuition:i<5?900:i<11?1200:1650,feeding:i<3?450:600,books:i<5?180:300,other:90}]))}return S.fees[y]}
fees(S.year);
const fee=(y,c)=>sum(Object.values(fees(y)[c]||{}));
const cls=(p,y)=>(S.enrol.find(e=>e.pid==p&&e.year==y)||{}).cls;
const nm=p=>(S.pupils.find(x=>x.id==p)||{}).name||'?';
const billed=(p,y)=>{const c=cls(p,y);return c?fee(y,c):0};
const paid=(p,y)=>sum(S.pay.filter(x=>x.pid==p&&(x.ay||yo(x.date))==y).map(x=>x.amt));
const yrsOf=p=>[...new Set(S.enrol.filter(e=>e.pid==p).map(e=>e.year))].sort();
const owed=p=>sum(yrsOf(p).map(y=>billed(p,y)-paid(p,y)));
const ssE=r=>r.gross*S.ssE/100,ssR=r=>r.gross*S.ssR/100,net=r=>r.gross-ssE(r)-(+r.paye||0);
const grade=a=>a>=80?'A':a>=70?'B':a>=60?'C':a>=50?'D':'E';
const stt=r=>r.paid?'Paid':r.due<today()?'OVERDUE':'Pending';
const scores=(p,y,t)=>S.res.filter(r=>r.pid==p&&r.year==y&&r.term==t).map(r=>+r.score);
const avgT=(p,y,t)=>{const v=scores(p,y,t);return v.length?(sum(v)/v.length).toFixed(1):'—'};
const pidL=()=>S.pupils.map(p=>[p.id,p.id+' · '+p.name]);
const stSel=(r,i)=>`<select onchange="S.pupils[${i}].status=this.value;save();render()">${['Active','Left','Graduated'].map(s=>`<option ${s==r.status?'selected':''}>${s}</option>`).join('')}</select>`;
const T={
pupils:{t:'Pupils — permanent ID, never reused',nd:1,q:['name'],f:[['name','Full name'],['dob','Date of birth','d0'],['gender','Gender','sel',['M','F']],['guardian','Guardian'],['phone','Phone'],['cls','Class this year','sel',CL]],
 p:r=>{r.id='P'+String(S.nid++).padStart(4,'0');r.status='Active';if(r.cls)S.enrol.push({year:S.year,pid:r.id,cls:r.cls});delete r.cls},
 c:[['ID',r=>`<b>${r.id}</b>`],['Name',r=>esc(r.name)],['Class now',r=>cls(r.id,S.year)||'—'],['Guardian',r=>esc(r.guardian)],['Owes (all years)',r=>fmt(owed(r.id))],['Status',stSel]]},
pay:{t:'Fees received',q:['pid','amt'],f:[['date','Date','date'],['pid','Pupil','sel',pidL],['item','For','sel',['Tuition','Feeding','Books','Other']],['amt','Amount','num'],['method','Method','sel',['Cash','MoMo','Bank']],['rc','Receipt no.'],['ay','Apply to year (optional)','sel',YRS]],
 c:[['Date',r=>r.date],['Pupil',r=>r.pid+' · '+esc(nm(r.pid))],['Year',r=>r.ay||yo(r.date)],['For',r=>r.item],['Amount',r=>fmt(r.amt)],['Method',r=>r.method],['Receipt',r=>esc(r.rc)]]},
exp:{t:'Bills & other expenses',q:['cat','amt'],f:[['date','Date','date'],['cat','Category','sel',CATS],['desc','Details'],['amt','Amount','num']],
 c:[['Date',r=>r.date],['Category',r=>r.cat],['Details',r=>esc(r.desc)],['Amount',r=>fmt(r.amt)]]},
staff:{t:'Staff register',q:['name','salary'],f:[['name','Full name'],['role','Role'],['salary','Monthly salary','num']],p:r=>{r.id='S'+String(S.sid++).padStart(3,'0')},
 c:[['ID',r=>`<b>${r.id}</b>`],['Name',r=>esc(r.name)],['Role',r=>esc(r.role)],['Salary',r=>fmt(r.salary)]]},
prl:{t:'Salary payments (SSNIT estimated from Setup rates)',q:['sid','gross'],f:[['date','Date','date'],['sid','Staff','sel',()=>S.staff.map(s=>[s.id,s.id+' · '+s.name])],['mo','Salary for (e.g. Sep 2026)'],['gross','Gross salary','num'],['paye','PAYE deducted','num']],
 c:[['Date',r=>r.date],['Staff',r=>esc((S.staff.find(s=>s.id==r.sid)||{}).name)],['Month',r=>esc(r.mo)],['Gross',r=>fmt(r.gross)],['SSNIT staff',r=>fmt(ssE(r))],['SSNIT school',r=>fmt(ssR(r))],['PAYE',r=>fmt(r.paye)],['Net pay',r=>fmt(net(r))]]},
stat:{t:'Statutory fees, taxes & permits',q:['type','amt','due'],f:[['due','Due date','date'],['type','Type','sel',STY],['desc','Details / period'],['amt','Amount due','num']],
 c:[['Due',r=>r.due],['Type',r=>r.type],['Details',r=>esc(r.desc)],['Amount',r=>fmt(r.amt)],['Status',r=>stt(r)=='OVERDUE'?'<span class="bad">OVERDUE</span>':stt(r)],['',(r,i)=>r.paid?'Paid '+r.paid+' · '+fmt(r.paidAmt):`<button class="btn s" onclick="payStat(${i})">Mark paid</button>`]]},
proj:{t:'School projects & expansion',q:['name','budget'],f:[['name','Project name'],['type','Type','sel',['Building','Expansion','Equipment','Renovation','Other']],['budget','Budget','num'],['status','Status','sel',['Planned','Ongoing','Completed','On hold']]],
 c:[['Project',r=>esc(r.name)],['Type',r=>r.type],['Budget',r=>fmt(r.budget)],['Received',r=>fmt(sum(S.led.filter(l=>l.proj==r.name&&l.type=='Contribution').map(l=>l.amt)))],['Spent',r=>fmt(sum(S.led.filter(l=>l.proj==r.name&&l.type=='Expense').map(l=>l.amt)))],['% used',r=>r.budget?Math.round(sum(S.led.filter(l=>l.proj==r.name&&l.type=='Expense').map(l=>l.amt))/r.budget*100)+'%':''],['Status',r=>r.status]]},
led:{t:'Project money in & out',q:['proj','type','amt'],f:[['date','Date','date'],['proj','Project','sel',()=>S.proj.map(p=>p.name)],['type','Type','sel',['Contribution','Expense']],['desc','Source / details'],['amt','Amount','num']],
 c:[['Date',r=>r.date],['Project',r=>esc(r.proj)],['Type',r=>r.type],['Details',r=>esc(r.desc)],['Amount',r=>fmt(r.amt)]]},
att:{t:'Attendance (per pupil per term)',q:['pid','open','present'],f:[['year','Year (blank = current)','sel',YRS],['term','Term','sel',['1','2','3']],['pid','Pupil','sel',pidL],['open','Days open','num'],['present','Days present','num']],
 p:r=>{r.year=r.year||S.year;r.term=r.term||S.term},c:[['Year',r=>r.year],['Term',r=>r.term],['Pupil',r=>r.pid+' · '+esc(nm(r.pid))],['Present / open',r=>r.present+' / '+r.open],['%',r=>r.open?Math.round(r.present/r.open*100)+'%':'']]},
res:{t:'Enter scores (out of 100)',q:['pid','subject','score'],f:[['term','Term (blank = current)','sel',['1','2','3']],['pid','Pupil','sel',pidL],['subject','Subject','sel',SUBJ],['score','Score','num']],
 p:r=>{r.year=S.year;r.term=r.term||S.term},c:[['Year',r=>r.year],['Term',r=>r.term],['Pupil',r=>r.pid+' · '+esc(nm(r.pid))],['Subject',r=>r.subject],['Score',r=>r.score]]}};
function inp(k,f){const[n,l,t,o]=f,id='f_'+k+'_'+n;
 if(t=='sel'){const op=(typeof o=='function'?o():o).map(x=>Array.isArray(x)?x:[x,x]);return`<label>${l}<select id="${id}"><option value="">—</option>${op.map(([v,x])=>`<option value="${esc(v)}">${esc(x)}</option>`).join('')}</select></label>`}
 return`<label>${l}<input id="${id}" type="${t=='num'?'number':t&&t[0]=='d'?'date':'text'}" step="any" ${t=='date'?`value="${today()}"`:''}></label>`}
function sect(k){const c=T[k],rows=S[k].map((r,i)=>[r,i]).reverse().slice(0,150);
 return`<div class="card"><h2>${c.t}</h2><div class="row">${c.f.map(f=>inp(k,f)).join('')}<button class="btn" onclick="add('${k}')">Add</button></div><div class="tw"><table><tr>${c.c.map(x=>`<th>${x[0]}`).join('')}${c.nd?'':'<th>'}</tr>${rows.map(([r,i])=>`<tr>${c.c.map(x=>`<td>${x[1](r,i)}`).join('')}${c.nd?'':`<td><button class="btn g s" onclick="del('${k}',${i})">✕</button>`}</tr>`).join('')||'<tr><td colspan="9">Nothing yet.</td></tr>'}</table></div></div>`}
function add(k){const c=T[k],r={};for(const[n,l,t]of c.f){const v=$('f_'+k+'_'+n).value;r[n]=t=='num'?(v===''?'':+v):v}
 if((c.q||[]).some(n=>r[n]===''||r[n]==null))return alert('Please fill in: '+c.q.join(', '));if(c.p)c.p(r);S[k].push(r);save();render()}
function del(k,i){if(confirm('Delete this entry?')){S[k].splice(i,1);save();render()}}
function payStat(i){const a=prompt('Amount paid (GH₵)',S.stat[i].amt);if(a===null)return;S.stat[i].paid=today();S.stat[i].paidAmt=+a;save();render()}
function promote(){const y=S.year,ny=shift(y,1);if(!confirm('Promote all active pupils to '+ny+' and make it the current year?'))return;
 S.enrol.filter(e=>e.year==y).forEach(e=>{const p=S.pupils.find(x=>x.id==e.pid);if(!p||p.status!='Active')return;const n=S.rep[e.pid]?e.cls:CL[CL.indexOf(e.cls)+1];
  if(!n)p.status='Graduated';else if(!cls(e.pid,ny))S.enrol.push({year:ny,pid:e.pid,cls:n})});
 S.year=ny;S.rep={};fees(ny);save();render()}

function dl(name,text,mime){const a=document.createElement('a');a.href=URL.createObjectURL(new Blob([text],{type:mime}));a.download=name;document.body.appendChild(a);a.click();a.remove();setTimeout(()=>URL.revokeObjectURL(a.href),1500)}
function backup(){S.lastBackup=Date.now();save(0);dl('school-backup-'+today()+'.json',JSON.stringify(S),'application/json');render()}
function restore(el){const f=el.files[0];if(!f)return;const r=new FileReader();r.onload=()=>{try{const d=JSON.parse(r.result);if(!d||!Array.isArray(d.pupils)||!d.fees)throw 0;if(!confirm('Replace ALL current data with this backup ('+d.pupils.length+' pupils)?'))return;S=d;fees(S.year);save(0);tab='dash';render()}catch(e){alert('That file is not a valid School Manager backup.')}};r.readAsText(f)}
function csv(k){let rows=S[k];if(k=='fees')rows=Object.entries(S.fees).flatMap(([y,c])=>Object.entries(c).map(([n,v])=>({year:y,class:n,...v})));
 const ks=[...new Set(rows.flatMap(Object.keys))],q=v=>'"'+String(v==null?'':v).replace(/"/g,'""')+'"';
 dl('school-'+k+'-'+today()+'.csv','\ufeff'+[ks.map(q).join(',')].concat(rows.map(r=>ks.map(x=>q(r[x])).join(','))).join('\r\n'),'text/csv')}
const bk=()=>{if(!S.pupils.length)return'';const d=S.lastBackup,n=d?Math.floor((Date.now()-d)/864e5):0,bad=!d||n>=7;return`<div class="card" ${bad?'style="border-color:var(--bad)"':''}>💾 ${d?'Last backup: '+new Date(d).toLocaleDateString()+' ('+n+' days ago)':'<b class="bad">No backup yet</b>'} · <a href="#" onclick="tab='backup';render();return false">Back up now</a></div>`};
const V={
dash(){const y=S.year,en=S.enrol.filter(e=>e.year==y),b=sum(en.map(e=>billed(e.pid,y))),p=sum(en.map(e=>paid(e.pid,y))),all=sum(S.pupils.map(x=>owed(x.id)));
 const iy=a=>a.filter(r=>yo(r.date)==y),led=t=>sum(iy(S.led).filter(r=>r.type==t).map(r=>r.amt)),
 rec=sum(iy(S.pay).map(r=>r.amt)),ex=sum(iy(S.exp).map(r=>r.amt)),np=sum(iy(S.prl).map(net)),sp=S.stat.filter(r=>r.paid&&yo(r.paid)==y),st=sum(sp.map(r=>r.paidAmt)),
 remit=sum(iy(S.prl).map(r=>ssE(r)+ssR(r)+(+r.paye||0)))-sum(sp.filter(r=>r.type=='SSNIT'||r.type=='PAYE').map(r=>r.paidAmt)),
 un=S.stat.filter(r=>!r.paid),cash=+S.open+rec+led('Contribution')-ex-np-st-led('Expense'),
 s=(l,v,c)=>`<div class="stat"><span>${l}</span><b class="${c||''}">${v}</b></div>`;
 const rows=CL.map(c=>{const e=en.filter(x=>x.cls==c),bb=sum(e.map(x=>billed(x.pid,y))),pp=sum(e.map(x=>paid(x.pid,y)));return`<tr><td>${c}<td>${e.length}<td>${fmt(bb)}<td>${fmt(pp)}<td class="${bb-pp>0?'bad':''}">${fmt(bb-pp)}</tr>`}).join('');
 return bk()+`<p class="mu">Academic year <b>${y}</b>, Term ${S.term}. Change it under Setup.</p><div class="grid">${s('Pupils enrolled',en.length)}${s('Fees billed',fmt(b))}${s('Fees paid (this year)',fmt(p))}${s('Owing this year',fmt(b-p),b-p>0?'bad':'')}${s('Arrears from earlier years',fmt(all-(b-p)))}${s('Total owed to school',fmt(all),'bad')}</div>
 <div class="grid">${s('Fees received',fmt(rec))}${s('Project contributions',fmt(led('Contribution')))}${s('Bills & expenses',fmt(ex))}${s('Staff net pay',fmt(np))}${s('Statutory & taxes paid',fmt(st))}${s('Project spending',fmt(led('Expense')))}${s('Cash balance',fmt(cash),cash<0?'bad':'')}</div>
 <div class="grid">${s('Statutory unpaid',fmt(sum(un.map(r=>r.amt))))}${s('OVERDUE items',S.stat.filter(r=>stt(r)=='OVERDUE').length,'bad')}${s('SSNIT + PAYE still to pay',fmt(remit))}</div>
 <div class="card"><h2>By class · ${y}</h2><div class="tw"><table><tr><th>Class<th>Pupils<th>Billed<th>Paid<th>Owing</tr>${rows}</table></div></div>`},
pupils:()=>sect('pupils'),pay:()=>sect('pay'),exp:()=>sect('exp'),stat:()=>sect('stat'),att:()=>sect('att'),
staff:()=>sect('staff')+sect('prl'),proj:()=>sect('proj')+sect('led'),
res(){const c=window.rc||CL[0],t=window.rt||S.term,y=S.year;
 const rk=S.enrol.filter(e=>e.year==y&&e.cls==c).map(e=>{const v=scores(e.pid,y,t);return{id:e.pid,n:v.length,a:v.length?sum(v)/v.length:null}}).filter(x=>x.a!==null).sort((a,b)=>b.a-a.a);
 return sect('res')+`<div class="card"><h2>Class positions</h2><div class="row"><select onchange="window.rc=this.value;render()">${CL.map(x=>`<option ${x==c?'selected':''}>${x}</option>`).join('')}</select><select onchange="window.rt=this.value;render()">${[1,2,3].map(x=>`<option value="${x}" ${x==t?'selected':''}>Term ${x}</option>`).join('')}</select></div><div class="tw"><table><tr><th>Pos<th>ID<th>Pupil<th>Subjects<th>Average<th>Grade</tr>${rk.map((x,i)=>`<tr><td>${i+1}<td>${x.id}<td>${esc(nm(x.id))}<td>${x.n}<td>${x.a.toFixed(1)}<td><span class="tag">${grade(x.a)}</span></tr>`).join('')||'<tr><td colspan="6">No scores for this class and term.</td></tr>'}</table></div></div>`},
promo(){const y=S.year,ny=shift(y,1),rows=S.enrol.filter(e=>e.year==y&&(S.pupils.find(p=>p.id==e.pid)||{}).status=='Active');
 const nx=e=>S.rep[e.pid]?e.cls:CL[CL.indexOf(e.cls)+1]||'Graduates';
 return`<div class="card"><h2>Year-end promotion: ${y} → ${ny}</h2><p class="mu">Tick "Repeat" for pupils staying in the same class. Everyone else moves up; JHS 3 pupils become Graduated. Old records, fees and results stay on file for every pupil.</p><button class="btn" onclick="promote()">Promote all to ${ny}</button></div>
 <div class="card"><div class="tw"><table><tr><th>ID<th>Name<th>Class now<th>Repeat<th>Next year</tr>${rows.map(e=>`<tr><td>${e.pid}<td>${esc(nm(e.pid))}<td>${e.cls}<td><input type="checkbox" ${S.rep[e.pid]?'checked':''} onchange="S.rep['${e.pid}']=this.checked;save();render()"><td><b>${nx(e)}</b></tr>`).join('')||'<tr><td colspan="5">No active pupils enrolled in this year.</td></tr>'}</table></div></div>`},
rec(){const id=window.rp||(S.pupils[0]||{}).id,p=S.pupils.find(x=>x.id==id);
 if(!p)return'<div class="card">Add a pupil first.</div>';
 const ys=yrsOf(id),pick=`<select onchange="window.rp=this.value;render()">${S.pupils.map(x=>`<option value="${x.id}" ${x.id==id?'selected':''}>${x.id} · ${esc(x.name)}</option>`).join('')}</select>`;
 return`<div class="card"><h2>Pupil record</h2><div class="row">${pick}</div><p><b>${esc(p.name)}</b> · ${p.id} · ${p.gender||''} · Born ${p.dob||'—'}<br><span class="mu">Guardian: ${esc(p.guardian)} ${esc(p.phone)} · Status: ${p.status}</span></p>
 <div class="tw"><table><tr><th>Year<th>Class<th>Billed<th>Paid<th>Balance<th>Avg T1<th>Avg T2<th>Avg T3<th>Attendance</tr>${ys.map(y=>{const a=S.att.filter(x=>x.pid==id&&x.year==y),o=sum(a.map(x=>x.open));return`<tr><td>${y}<td>${cls(id,y)}<td>${fmt(billed(id,y))}<td>${fmt(paid(id,y))}<td class="${billed(id,y)-paid(id,y)>0?'bad':''}">${fmt(billed(id,y)-paid(id,y))}<td>${avgT(id,y,1)}<td>${avgT(id,y,2)}<td>${avgT(id,y,3)}<td>${o?Math.round(sum(a.map(x=>x.present))/o*100)+'%':'—'}</tr>`}).join('')||'<tr><td colspan="9">Not enrolled in any year yet.</td></tr>'}</table></div><p><b>Total owed over school life: ${fmt(owed(id))}</b></p></div>`},
backup(){const L=[['pupils','Pupils'],['enrol','Enrolment (class history)'],['fees','Fees'],['pay','Payments'],['exp','Expenses'],['staff','Staff'],['prl','Salary payments'],['stat','Statutory'],['proj','Projects'],['led','Project ledger'],['att','Attendance'],['res','Results']];
 return`<div class="card"><h2>Backup & restore</h2><p class="mu">Your data is stored inside this browser on this computer. Clearing browser data would erase it — so download a backup regularly (weekly is good) and keep the file on a flash drive or email it to yourself.</p><div class="row"><button class="btn" onclick="backup()">⬇ Download backup (.json)</button><label>Restore from a backup file<input type="file" accept=".json,application/json" onchange="restore(this)"></label></div><p class="mu">${S.lastBackup?'Last backup: '+new Date(S.lastBackup).toLocaleString():'No backup yet.'} Restoring replaces everything currently here.</p></div>
 <div class="card"><h2>Export to Excel (CSV)</h2><p class="mu">One file per table; opens directly in Excel. Use this for sharing or printing — use the .json backup for restoring.</p><div class="row">${L.map(([k,l])=>`<button class="btn g s" style="color:var(--tx)" onclick="csv('${k}')">${l}</button>`).join('')}</div></div>
 <div class="card"><h2>Using it offline</h2><p class="mu">This file needs no internet. Keep it on your computer and double-click it to open it in your browser, always the same file and the same browser, so it finds your saved data. A backup also lets you move to another computer: open the file there and use Restore.</p></div>`},
setup(){const f=fees(S.year);
 return`<div class="card"><h2>Current year & rates</h2><div class="row"><label>Academic year<select onchange="S.year=this.value;fees(S.year);save();render()">${YRS.map(y=>`<option ${y==S.year?'selected':''}>${y}</option>`).join('')}</select></label><label>Term<select onchange="S.term=+this.value;save();render()">${[1,2,3].map(t=>`<option ${t==S.term?'selected':''}>${t}</option>`).join('')}</select></label>
 <label>SSNIT staff %<input class="fi" type="number" step="0.1" value="${S.ssE}" onchange="S.ssE=+this.value;save()"></label><label>SSNIT school %<input class="fi" type="number" step="0.1" value="${S.ssR}" onchange="S.ssR=+this.value;save()"></label><label>Opening cash<input class="fi" type="number" value="${S.open}" onchange="S.open=+this.value;save();render()"></label></div><p class="mu">SSNIT 5.5% (employee) and 13% (employer) are the standard rates; PAYE is entered by you.</p></div>
 <div class="card"><h2>Fees per pupil for ${S.year} (whole year)</h2><p class="mu">Sample amounts — change them. A new year copies last year's fees, so old bills never change.</p><div class="tw"><table><tr><th>Class<th>Tuition<th>Feeding<th>Books<th>Other<th>Total</tr>${CL.map(c=>`<tr><td>${c}${['tuition','feeding','books','other'].map(k=>`<td><input class="fi" type="number" value="${f[c][k]}" onchange="fees(S.year)['${c}'].${k}=+this.value;save();render()">`).join('')}<td><b>${fmt(fee(S.year,c))}</b></tr>`).join('')}</table></div></div>
 <div class="card"><button class="btn g" onclick="if(confirm('Delete ALL data?')){localStorage.removeItem(KEY);location.reload()}">Reset all data</button></div>`}};
const TABS=[['dash','Overview'],['pupils','Pupils'],['rec','Pupil record'],['pay','Payments'],['exp','Expenses'],['staff','Staff & pay'],['stat','Statutory'],['proj','Projects'],['att','Attendance'],['res','Results'],['promo','Promotion'],['backup','Backup'],['setup','Setup']];
let tab='dash';
function render(){$('nav').innerHTML=TABS.map(([k,l])=>`<button class="${k==tab?'on':''}" onclick="tab='${k}';render()">${l}</button>`).join('');$('app').innerHTML=V[tab]()}
render();
</script></body></html>
