<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Tasks</title>
<link href="https://fonts.googleapis.com/css2?family=Figtree:wght@400;500;700;800&display=swap" rel="stylesheet">
<style>
:root{--bg:#eef2f7;--card:#fff;--ink:#16202e;--mut:#66758a;--line:#dbe3ee;--acc:#2f5fd0;--acc-t:#fff;--hi:#d6453d;--md:#c98a00;--lo:#2b8a5f;
box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#0f1621;--card:#182231;--ink:#e7edf6;--mut:#8b9bb1;--line:#263449;--acc:#6f95ff;--acc-t:#0f1621;--hi:#ff7a70;--md:#e6b23a;--lo:#5cc596}}
:root[data-theme="dark"]{--bg:#0f1621;--card:#182231;--ink:#e7edf6;--mut:#8b9bb1;--line:#263449;--acc:#6f95ff;--acc-t:#0f1621;--hi:#ff7a70;--md:#e6b23a;--lo:#5cc596}
html{scroll-padding-top:env(safe-area-inset-top,0px)}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--ink);font:16px/1.45 Figtree,system-ui,sans-serif}
main{max-width:640px;margin:0 auto;padding:24px 16px 48px}
h1{font-size:2rem;font-weight:800;margin:0 0 4px;letter-spacing:-.02em}
.sub{color:var(--mut);margin:0 0 16px}
.bar{height:8px;background:var(--line);border-radius:8px;overflow:hidden;margin-bottom:20px}
.bar i{display:block;height:100%;width:0;background:var(--lo);transition:width .3s}
form,.item{background:var(--card);border:1px solid var(--line);border-radius:12px;padding:12px}
form{display:grid;gap:8px;margin-bottom:16px}
.row{display:flex;gap:8px;flex-wrap:wrap}
input,select,textarea,button{font:inherit;color:var(--ink)}
input,select,textarea{background:var(--bg);border:1px solid var(--line);border-radius:8px;padding:9px 10px;min-width:0}
input[type=text]{flex:1 1 200px}
textarea{width:100%;resize:vertical;min-height:64px}
button{cursor:pointer;border:1px solid var(--line);background:var(--card);border-radius:8px;padding:8px 12px}
button.p{background:var(--acc);color:var(--acc-t);border-color:var(--acc);font-weight:700}
:focus-visible{outline:2px solid var(--acc);outline-offset:2px}
.tools{display:flex;gap:8px;flex-wrap:wrap;margin-bottom:12px}
.tools input{flex:1 1 140px}
.tabs{display:flex;gap:4px}
.tabs button[aria-pressed=true]{background:var(--ink);color:var(--bg);border-color:var(--ink)}
ul{list-style:none;margin:0;padding:0;display:grid;gap:10px}
.item{display:grid;grid-template-columns:auto 1fr auto;gap:10px;align-items:start;border-left:5px solid var(--lo)}
.item.high{border-left-color:var(--hi)}.item.medium{border-left-color:var(--md)}
.item input[type=checkbox]{width:22px;height:22px;margin-top:2px;accent-color:var(--acc)}
.t{font-weight:500;overflow-wrap:anywhere}
.done .t{text-decoration:line-through;color:var(--mut)}
.meta{font-size:.85rem;color:var(--mut);display:flex;gap:10px;flex-wrap:wrap;margin-top:2px}
.late{color:var(--hi);font-weight:700}
.note{margin-top:8px;padding:8px 10px;background:var(--bg);border-radius:8px;font-size:.92rem;white-space:pre-wrap;overflow-wrap:anywhere}
.acts{display:flex;gap:4px}
.acts button{padding:4px 9px;font-size:.85rem}
.edit{grid-column:1/-1;display:grid;gap:8px}
.empty{text-align:center;color:var(--mut);padding:28px 0}
</style>
</head>
<body>
<main>
<h1>Tasks</h1>
<p class="sub" id="sub"></p>
<div class="bar"><i id="bar"></i></div>

<form id="add">
  <div class="row">
    <input type="text" id="title" placeholder="What needs doing?" aria-label="Task title" required>
    <button class="p" type="submit">Add task</button>
  </div>
  <div class="row">
    <select id="pri" aria-label="Priority"><option value="low">Low priority</option><option value="medium" selected>Medium priority</option><option value="high">High priority</option></select>
    <input type="date" id="due" aria-label="Due date">
  </div>
  <textarea id="notes" placeholder="Notes (optional)" aria-label="Notes"></textarea>
</form>

<div class="tools">
  <input type="search" id="q" placeholder="Search tasks and notes" aria-label="Search">
  <div class="tabs" id="tabs">
    <button data-f="all" aria-pressed="true">All</button>
    <button data-f="active" aria-pressed="false">Active</button>
    <button data-f="done" aria-pressed="false">Done</button>
  </div>
  <select id="sort" aria-label="Sort by"><option value="new">Newest</option><option value="due">Due date</option><option value="pri">Priority</option></select>
</div>

<ul id="list"></ul>
<div class="row" style="margin-top:14px"><button id="clear">Clear completed</button></div>
</main>

<script>
var KEY='todo-app-v1', tasks=[], filter='all', editing=null;
try{ tasks=JSON.parse(localStorage.getItem(KEY)||'[]'); if(!Array.isArray(tasks)) tasks=[]; }catch(e){ tasks=[]; }
function save(){ try{ localStorage.setItem(KEY,JSON.stringify(tasks)); }catch(e){} }
var $=function(id){return document.getElementById(id)};
function esc(s){var d=document.createElement('div');d.textContent=s;return d.innerHTML}
var rank={high:0,medium:1,low:2};
function today(){var d=new Date();return d.getFullYear()+'-'+String(d.getMonth()+1).padStart(2,'0')+'-'+String(d.getDate()).padStart(2,'0')}
function fmt(s){var p=s.split('-');return new Date(p[0],p[1]-1,p[2]).toLocaleDateString(undefined,{month:'short',day:'numeric'})}

$('add').addEventListener('submit',function(e){
  e.preventDefault();
  var t=$('title').value.trim(); if(!t) return;
  tasks.unshift({id:Date.now(),title:t,pri:$('pri').value,due:$('due').value,notes:$('notes').value.trim(),done:false});
  $('title').value='';$('notes').value='';$('due').value='';
  save();render();$('title').focus();
});
$('tabs').addEventListener('click',function(e){
  var f=e.target.dataset.f; if(!f) return; filter=f;
  [].forEach.call($('tabs').children,function(b){b.setAttribute('aria-pressed',b.dataset.f===f)});
  render();
});
$('q').addEventListener('input',render);
$('sort').addEventListener('change',render);
$('clear').addEventListener('click',function(){tasks=tasks.filter(function(t){return !t.done});save();render()});

$('list').addEventListener('click',function(e){
  var li=e.target.closest('li'); if(!li) return;
  var id=+li.dataset.id, t=tasks.find(function(x){return x.id===id}), a=e.target.dataset.a;
  if(e.target.type==='checkbox'){t.done=e.target.checked;save();render();return}
  if(a==='del'){tasks=tasks.filter(function(x){return x.id!==id});save();render()}
  if(a==='edit'){editing=id;render()}
  if(a==='cancel'){editing=null;render()}
  if(a==='save'){
    var v=li.querySelector('.et').value.trim(); if(!v) return;
    t.title=v;t.pri=li.querySelector('.ep').value;t.due=li.querySelector('.ed').value;t.notes=li.querySelector('.en').value.trim();
    editing=null;save();render();
  }
});

function render(){
  var q=$('q').value.toLowerCase(), s=$('sort').value;
  var list=tasks.filter(function(t){
    if(filter==='active'&&t.done) return false;
    if(filter==='done'&&!t.done) return false;
    return !q||(t.title+' '+t.notes).toLowerCase().indexOf(q)>-1;
  });
  if(s==='due') list.sort(function(a,b){return (a.due||'9999')<(b.due||'9999')?-1:1});
  if(s==='pri') list.sort(function(a,b){return rank[a.pri]-rank[b.pri]});
  var done=tasks.filter(function(t){return t.done}).length, n=tasks.length;
  $('sub').textContent=n?done+' of '+n+' done':'Nothing here yet.';
  $('bar').style.width=n?(done/n*100)+'%':'0';
  $('list').innerHTML=list.length?list.map(row).join(''):'<li class="empty">'+(n?'No tasks match this view.':'Add your first task above.')+'</li>';
}
function row(t){
  if(t.id===editing){
    return '<li class="item '+t.pri+'" data-id="'+t.id+'"><div class="edit" style="grid-column:1/-1">'+
    '<input type="text" class="et" value="'+esc(t.title).replace(/"/g,'&quot;')+'" aria-label="Edit title">'+
    '<div class="row"><select class="ep" aria-label="Priority">'+['low','medium','high'].map(function(p){return '<option value="'+p+'"'+(p===t.pri?' selected':'')+'>'+p[0].toUpperCase()+p.slice(1)+' priority</option>'}).join('')+'</select>'+
    '<input type="date" class="ed" value="'+t.due+'" aria-label="Due date"></div>'+
    '<textarea class="en" aria-label="Notes">'+esc(t.notes)+'</textarea>'+
    '<div class="row"><button class="p" data-a="save">Save changes</button><button data-a="cancel">Cancel</button></div></div></li>';
  }
  var late=t.due&&!t.done&&t.due<today();
  return '<li class="item '+t.pri+(t.done?' done':'')+'" data-id="'+t.id+'">'+
  '<input type="checkbox" '+(t.done?'checked':'')+' aria-label="Mark complete">'+
  '<div><div class="t">'+esc(t.title)+'</div><div class="meta"><span>'+t.pri[0].toUpperCase()+t.pri.slice(1)+' priority</span>'+
  (t.due?'<span class="'+(late?'late':'')+'">'+(late?'Overdue: ':'Due ')+fmt(t.due)+'</span>':'')+'</div>'+
  (t.notes?'<div class="note">'+esc(t.notes)+'</div>':'')+'</div>'+
  '<div class="acts"><button data-a="edit">Edit</button><button data-a="del" aria-label="Delete task">Delete</button></div></li>';
}
render();
</script>
</body>
</html>