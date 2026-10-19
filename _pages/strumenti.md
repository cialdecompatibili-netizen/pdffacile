---
layout: single
title: "Strumenti PDF gratuiti"
permalink: /strumenti/
author_profile: false
seo_title: "Strumenti PDF gratuiti: unisci, dividi, ruota, da immagini a PDF | PDFFacile"
seo_description: "Unisci, dividi e ruota PDF e trasforma immagini in PDF direttamente nel browser, gratis e senza caricare i file."
classes: wide
---

<style>
.pdft{display:grid;grid-template-columns:repeat(auto-fit,minmax(300px,1fr));gap:18px;margin:1rem 0 2rem}
.pdft section{background:#fff;color:#1f2933;border:1px solid #e6e8ec;border-radius:18px;padding:18px 20px;box-shadow:0 1px 2px rgba(16,24,40,.04),0 8px 24px -12px rgba(16,24,40,.12)}
.pdft h2{margin:0 0 6px;font-size:1.15rem;color:#111827}
.pdft p{margin:0 0 10px;font-size:.93rem;color:#3a4552}
.pdft input[type=file],.pdft input[type=text],.pdft select{width:100%;margin:6px 0;box-sizing:border-box}
.pdft button{padding:.5rem 1.2rem;border-radius:999px;border:1px solid #111827;background:#111827;color:#fff;font-weight:600;cursor:pointer;margin-top:6px}
.pdft button:disabled{opacity:.5;cursor:default}
.pdft .esito{font-size:.9rem;margin-top:8px;min-height:1.2em}
.pdft .esito a{font-weight:600}
</style>

I file restano sul tuo computer: l'elaborazione avviene nel browser, niente viene caricato. Per file molto grandi, elaborazioni in serie e senza pubblicita' c'e' il [piano Pro]({{ '/servizi/' | relative_url }}).

<div class="pdft">
<section id="t-unisci"><h2>Unisci PDF</h2><p>Scegli due o piu' PDF: vengono uniti nell'ordine in cui li selezioni.</p>
<input type="file" id="u-f" accept="application/pdf" multiple><button id="u-b">Unisci</button><div class="esito" id="u-e"></div></section>

<section id="t-dividi"><h2>Estrai pagine</h2><p>Indica le pagine da tenere, per esempio <code>1-3,5,8-10</code>.</p>
<input type="file" id="d-f" accept="application/pdf"><input type="text" id="d-p" placeholder="1-3,5"><button id="d-b">Estrai</button><div class="esito" id="d-e"></div></section>

<section id="t-ruota"><h2>Ruota PDF</h2><p>Ruota tutte le pagine di un PDF.</p>
<input type="file" id="r-f" accept="application/pdf"><select id="r-g"><option value="90">90 gradi a destra</option><option value="180">180 gradi</option><option value="270">90 gradi a sinistra</option></select><button id="r-b">Ruota</button><div class="esito" id="r-e"></div></section>

<section id="t-img"><h2>Da immagini a PDF</h2><p>Scegli foto JPG o PNG: ogni immagine diventa una pagina.</p>
<input type="file" id="i-f" accept="image/jpeg,image/png" multiple><button id="i-b">Crea PDF</button><div class="esito" id="i-e"></div></section>
</div>

<script src="https://cdn.jsdelivr.net/npm/pdf-lib@1.17.1/dist/pdf-lib.min.js"></script>
<script>
(function(){
  var L=window.PDFLib, MAX=60*1024*1024;
  function $(i){return document.getElementById(i);}
  function buf(f){return f.arrayBuffer();}
  function ok(el,bytes,nome){
    var u=URL.createObjectURL(new Blob([bytes],{type:'application/pdf'}));
    el.innerHTML='Fatto. <a href="'+u+'" download="'+nome+'">Scarica '+nome+'</a>';
  }
  function err(el,e){el.textContent='Errore: '+(e&&e.message?e.message:e);}
  function check(files,el){
    for(var i=0;i<files.length;i++){ if(files[i].size>MAX){ el.textContent='File troppo grande (limite gratuito 60 MB). Con il piano Pro fino a 200 MB.'; return false; } }
    return true;
  }
  function run(btn,el,fn){
    btn.addEventListener('click',function(){
      el.textContent='Elaborazione...'; btn.disabled=true;
      Promise.resolve().then(fn).catch(function(e){err(el,e);}).then(function(){btn.disabled=false;});
    });
  }
  run($('u-b'),$('u-e'),function(){
    var fs=$('u-f').files; if(fs.length<2){$('u-e').textContent='Scegli almeno due PDF.';return;} if(!check(fs,$('u-e')))return;
    return L.PDFDocument.create().then(function(out){
      var p=Promise.resolve();
      Array.prototype.forEach.call(fs,function(f){ p=p.then(function(){return buf(f);}).then(function(b){return L.PDFDocument.load(b);}).then(function(d){return out.copyPages(d,d.getPageIndices());}).then(function(pg){pg.forEach(function(x){out.addPage(x);});}); });
      return p.then(function(){return out.save();});
    }).then(function(b){ok($('u-e'),b,'unito.pdf');});
  });
  function parse(s,n){
    var set=[],parts=s.replace(/\s/g,'').split(',');
    parts.forEach(function(t){ if(!t)return; var m=t.match(/^(\d+)(?:-(\d+))?$/); if(!m)throw new Error('Intervallo non valido: '+t);
      var a=+m[1],b=m[2]?+m[2]:a; if(a<1||b<a||b>n)throw new Error('Pagine fuori intervallo (il PDF ha '+n+' pagine)'); for(var i=a;i<=b;i++)set.push(i-1); });
    if(!set.length)throw new Error('Indica almeno una pagina'); return set;
  }
  run($('d-b'),$('d-e'),function(){
    var f=$('d-f').files[0]; if(!f){$('d-e').textContent='Scegli un PDF.';return;} if(!check([f],$('d-e')))return;
    return buf(f).then(function(b){return L.PDFDocument.load(b);}).then(function(d){
      var idx=parse($('d-p').value,d.getPageCount());
      return L.PDFDocument.create().then(function(out){return out.copyPages(d,idx).then(function(pg){pg.forEach(function(x){out.addPage(x);});return out.save();});});
    }).then(function(b){ok($('d-e'),b,'estratto.pdf');});
  });
  run($('r-b'),$('r-e'),function(){
    var f=$('r-f').files[0]; if(!f){$('r-e').textContent='Scegli un PDF.';return;} if(!check([f],$('r-e')))return;
    var g=+$('r-g').value;
    return buf(f).then(function(b){return L.PDFDocument.load(b);}).then(function(d){
      d.getPages().forEach(function(p){p.setRotation(L.degrees((p.getRotation().angle+g)%360));}); return d.save();
    }).then(function(b){ok($('r-e'),b,'ruotato.pdf');});
  });
  run($('i-b'),$('i-e'),function(){
    var fs=$('i-f').files; if(!fs.length){$('i-e').textContent='Scegli almeno un\'immagine.';return;} if(!check(fs,$('i-e')))return;
    return L.PDFDocument.create().then(function(out){
      var p=Promise.resolve();
      Array.prototype.forEach.call(fs,function(f){ p=p.then(function(){return buf(f);}).then(function(b){return /png$/i.test(f.type)?out.embedPng(b):out.embedJpg(b);}).then(function(im){var pg=out.addPage([im.width,im.height]);pg.drawImage(im,{x:0,y:0,width:im.width,height:im.height});}); });
      return p.then(function(){return out.save();});
    }).then(function(b){ok($('i-e'),b,'immagini.pdf');});
  });
})();
</script>
