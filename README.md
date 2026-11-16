# data-pasien-srimaidona
<!doctype html>
<html lang="id">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Pencatatan Data Pasien</title>
<style>
:root{--bg:#f4f7f9;--card:#fff;--tx:#17212b;--mu:#5d6d7c;--bd:#d8e0e7;--ac:#0f766e;--on:#fff;--dg:#b42318;--ok:#0f766e;color-scheme:light;box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#0e1418;--card:#161e24;--tx:#e8eef3;--mu:#9aabb8;--bd:#2a3640;--ac:#2dd4bf;--on:#06201c;--dg:#f97066;--ok:#2dd4bf;color-scheme:dark}}
:root[data-theme="dark"]{--bg:#0e1418;--card:#161e24;--tx:#e8eef3;--mu:#9aabb8;--bd:#2a3640;--ac:#2dd4bf;--on:#06201c;--dg:#f97066;--ok:#2dd4bf;color-scheme:dark}
*{box-sizing:border-box}
html{scroll-padding-top:env(safe-area-inset-top,0px)}
body{margin:0;background:var(--bg);color:var(--tx);font:16px/1.5 system-ui,-apple-system,"Segoe UI",Roboto,sans-serif}
.wrap{max-width:960px;margin:0 auto;padding:20px 16px 40px}
h1{font-size:1.4rem;margin:0 0 2px}
.sub{color:var(--mu);font-size:.9rem;margin:0 0 16px}
.badge{display:inline-block;font-size:.78rem;padding:2px 10px;border-radius:99px;border:1px solid var(--bd);color:var(--ok);margin-left:6px;vertical-align:middle}
.badge.warn{color:var(--dg)}
.card{background:var(--card);border:1px solid var(--bd);border-radius:12px;padding:16px;margin-bottom:16px}
.grid{display:grid;grid-template-columns:1fr 1fr;gap:12px}
.full{grid-column:1/-1}
@media(max-width:600px){.grid{grid-template-columns:1fr}}
label{display:block;font-size:.85rem;font-weight:600;margin-bottom:4px}
input,textarea{width:100%;padding:10px 12px;border:1px solid var(--bd);border-radius:8px;background:var(--bg);color:var(--tx);font:inherit}
input:focus,textarea:focus{outline:2px solid var(--ac);outline-offset:1px}
textarea{min-height:76px;resize:vertical}
.row{display:flex;gap:8px;flex-wrap:wrap;margin-top:14px}
button{font:inherit;padding:9px 16px;border-radius:8px;border:1px solid var(--bd);background:var(--card);color:var(--tx);cursor:pointer}
button.p{background:var(--ac);color:var(--on);border-color:var(--ac);font-weight:600}
button.d{color:var(--dg)}
.bar{display:flex;gap:12px;align-items:center;justify-content:space-between;flex-wrap:wrap;margin-bottom:12px}
.bar input{max-width:280px}
.item{border-top:1px solid var(--bd);padding:14px 0}
.item:first-child{border-top:0}
.top{display:flex;justify-content:space-between;gap:8px;align-items:flex-start;flex-wrap:wrap}
.nm{font-weight:700;font-size:1.05rem}
.meta{color:var(--mu);font-size:.85rem}
dl{display:grid;grid-template-columns:130px 1fr;gap:4px 12px;margin:8px 0 0;font-size:.92rem}
dt{color:var(--mu)}dd{margin:0;white-space:pre-wrap;overflow-wrap:anywhere}
@media(max-width:600px){dl{grid-template-columns:1fr}dt{margin-top:6px}}
.empty{color:var(--mu);text-align:center;padding:24px 0}
#msg{font-size:.88rem;color:var(--dg);min-height:1.2em;margin-top:8px}
</style>
</head>
<body>
<div class="wrap">
  <h1>Pencatatan Data Pasien <span id="mode" class="badge">memuat…</span></h1>
  <p class="sub">Catat data pasien, hasil koreksi, frame, dan lensa. Data tersimpan di database.</p>

  <div class="card">
    <div class="grid">
      <div><label for="nama">Nama</label><input id="nama" autocomplete="off"></div>
      <div><label for="umur">Umur (tahun)</label><input id="umur" type="number" min="0" max="120" inputmode="numeric"></div>
      <div class="full"><label for="anam">Anamnesa</label><textarea id="anam" placeholder="Keluhan, riwayat penyakit, kebiasaan, dll."></textarea></div>
      <div class="full"><label for="kor">Hasil koreksi</label><textarea id="kor" placeholder="Contoh: OD S -1.00 C -0.50 x 180 | OS S -1.25 | Add +1.50"></textarea></div>
      <div><label for="frame">Jenis Frame</label><input id="frame" list="fl" autocomplete="off"><datalist id="fl"><option>Full frame plastik</option><option>Full frame metal</option><option>Half frame</option><option>Rimless</option><option>Titanium</option></datalist></div>
      <div><label for="lensa">Jenis Lensa</label><input id="lensa" list="ll" autocomplete="off"><datalist id="ll"><option>Single vision</option><option>Bifokal</option><option>Progresif</option><option>Blue-cut</option><option>Photochromic</option></datalist></div>
    </div>
    <div class="row">
      <button class="p" id="save">Simpan</button>
      <button id="reset" hidden>Batal edit</button>
    </div>
    <div id="msg" role="status"></div>
  </div>

  <div class="card">
    <div class="bar"><strong id="count">Daftar Pasien</strong><input id="q" type="search" placeholder="Cari nama…"></div>
    <div id="list"></div>
  </div>
</div>

<script>
(function(){
  var $=function(i){return document.getElementById(i)};
  var rows=[],editId=null,store=null,LS='pasien_optik_v1',loaded=false;

  function esc(s){return String(s==null?'':s).replace(/[&<>"']/g,function(c){return{'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]})}
  function msg(t){$('msg').textContent=t||''}
  function setMode(t,warn){var m=$('mode');m.textContent=t;m.className='badge'+(warn?' warn':'')}
  function lsLoad(){try{return JSON.parse(localStorage.getItem(LS)||'[]')}catch(e){return[]}}
  function lsSave(){try{localStorage.setItem(LS,JSON.stringify(rows))}catch(e){}}

  function render(){
    var q=$('q').value.trim().toLowerCase();
    var list=rows.filter(function(r){return !q||String(r.nama||'').toLowerCase().indexOf(q)>-1})
      .sort(function(a,b){return (b.createdAt||0)-(a.createdAt||0)});
    $('count').textContent='Daftar Pasien ('+rows.length+')';
    if(!list.length){$('list').innerHTML='<div class="empty">'+(rows.length?'Tidak ada hasil.':'Belum ada data pasien.')+'</div>';return}
    $('list').innerHTML=list.map(function(r){
      var tgl=r.createdAt?new Date(r.createdAt).toLocaleDateString('id-ID',{day:'numeric',month:'short',year:'numeric'}):'';
      return '<div class="item"><div class="top"><div><span class="nm">'+esc(r.nama)+'</span> <span class="meta">· '+esc(r.umur)+' th'+(tgl?' · '+tgl:'')+'</span></div>'+
        '<div><button data-e="'+esc(r.id)+'">Edit</button> <button class="d" data-d="'+esc(r.id)+'">Hapus</button></div></div>'+
        '<dl><dt>Anamnesa</dt><dd>'+esc(r.anamnesa)+'</dd><dt>Hasil koreksi</dt><dd>'+esc(r.koreksi)+'</dd><dt>Jenis Frame</dt><dd>'+esc(r.frame)+'</dd><dt>Jenis Lensa</dt><dd>'+esc(r.lensa)+'</dd></dl></div>';
    }).join('');
  }

  function toRows(snap){
    var out=[];
    function each(d){var v=typeof d.data==='function'?d.data():d;out.push(Object.assign({},v,{id:d.id||v.id}))}
    if(Array.isArray(snap))snap.forEach(each);
    else if(snap&&snap.docs)snap.docs.forEach(each);
    else if(snap&&snap.forEach)snap.forEach(each);
    return out;
  }

  function fallback(){
    if(loaded)return;loaded=true;
    rows=lsLoad();setMode('tersimpan di peramban saja',true);render();
  }

  async function init(){
    var timer=setTimeout(fallback,10000);
    try{
      if(window.claude&&claude.use){
        var user=await claude.use('user'),db=await claude.use('db');
        if(user&&db){
          var uid=await user.id();
          store=db.collection('data/users/'+uid);
          store.onSnapshot(function(snap){
            clearTimeout(timer);loaded=true;rows=toRows(snap);
            setMode('terhubung ke database');render();
          },function(){clearTimeout(timer);store=null;fallback()});
          return;
        }
      }
    }catch(e){}
    clearTimeout(timer);fallback();
  }

  function clearForm(){
    ['nama','umur','anam','kor','frame','lensa'].forEach(function(i){$(i).value=''});
    editId=null;$('save').textContent='Simpan';$('reset').hidden=true;
  }

  $('save').onclick=async function(){
    var nama=$('nama').value.trim(),umur=$('umur').value.trim();
    if(!nama||!umur){msg('Nama dan umur wajib diisi.');return}
    var old=rows.filter(function(r){return r.id===editId})[0];
    var id=editId||('p'+Date.now());
    var rec={id:id,nama:nama,umur:Number(umur),anamnesa:$('anam').value.trim(),koreksi:$('kor').value.trim(),
      frame:$('frame').value.trim(),lensa:$('lensa').value.trim(),createdAt:old?old.createdAt:Date.now(),updatedAt:Date.now()};
    try{
      if(store){await store.doc(id).set(rec)}
      else{rows=rows.filter(function(r){return r.id!==id});rows.push(rec);lsSave();render()}
      msg('');clearForm();
    }catch(e){msg('Gagal menyimpan: '+(e&&e.message?e.message:'coba lagi'))}
  };
  $('reset').onclick=function(){clearForm();msg('')};
  $('q').oninput=render;

  $('list').onclick=async function(ev){
    var b=ev.target.closest('button');if(!b)return;
    var eid=b.getAttribute('data-e'),did=b.getAttribute('data-d');
    if(eid){
      var r=rows.filter(function(x){return x.id===eid})[0];if(!r)return;
      editId=eid;$('nama').value=r.nama||'';$('umur').value=r.umur||'';$('anam').value=r.anamnesa||'';
      $('kor').value=r.koreksi||'';$('frame').value=r.frame||'';$('lensa').value=r.lensa||'';
      $('save').textContent='Perbarui';$('reset').hidden=false;window.scrollTo({top:0,behavior:'smooth'});
    }else if(did){
      if(!confirm('Hapus data pasien ini?'))return;
      try{
        if(store){await store.doc(did).delete()}
        else{rows=rows.filter(function(x){return x.id!==did});lsSave();render()}
      }catch(e){msg('Gagal menghapus: '+(e&&e.message?e.message:'coba lagi'))}
    }
  };

  render();init();
})();
</script>
</body>
</html>
