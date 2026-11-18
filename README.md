return `<div class=card><div>Faturado hoje</div><div class=tot>${R(f(v=>d(v.t)==h))}</div><div>No mês: <b>${R(mes)}</b> · Boletos em aberto: <b>${R(bp)}</b></div>
<small>${PG.map(p=>p+' '+R(f(v=>v.pg==p&&d(v.t).slice(0,7)==h.slice(0,7)))).join(' · ')}</small></div>
<div class=card><b>Boletos</b><div class=row><input id=bd placeholder=Descrição><input id=bv type=number step=.01 placeholder=Valor><input id=bt type=date><button class=g onclick="ab()">+</button></div>
${S.bol.map((b,i)=>`<div class=row><span>${E(b.d)} · ${R(b.val)} · ${E(b.v)}</span><button class=s onclick="S.bol[${i}].pago=!S.bol[${i}].pago;save();draw()">${b.pago?'Pago ✓':'Pagar'}</button><button class=s onclick="S.bol.splice(${i},1);save();draw()">🗑</button></div>`).join('')}</div>
${per()}${nota()}
<div class=card><b>QR das mesas</b><div class=row><input id=qn type=number placeholder="Nº da mesa"><button class=g onclick="qr()">Gerar</button></div><div id=qc style="background:#fff;padding:12px;display:none;width:fit-content"></div></div>
<div class=card><b>Alterar senha do dono</b><div class=row><input id=pwa type=password placeholder="Senha atual"></div><div class=row><input id=pwn type=password placeholder="Nova senha"></div><div class=row><input id=pwr type=password placeholder="Repita a nova senha"><button class=g onclick="trocaSenha()">Salvar</button></div></div>`}
let NT={it:[],mg:'',est:false,busy:false,err:''},PR={a:'',b:''};
const nm=s=>String(s||'').toLowerCase().normalize('NFD').replace(/[\u0300-\u036f]/g,'').replace(/[^a-z0-9]/g,'');
function mt(x){const c=String(x.codigo||'').trim();if(c&&S.prods[c])return c;const n=nm(x.nome);if(n.length<4)return '';return Object.keys(S.prods).find(k=>{const m=nm(S.prods[k].n);return m&&(m==n||m.includes(n)||n.includes(m))})||''}
function rc(){const m=parseFloat(NT.mg);if(isNaN(m))return;NT.it.forEach(x=>{if(x.c)x.pv=Math.round(x.cu*(1+m/100)*100)/100})}
function rz(f){return new Promise((ok,no)=>{const i=new Image();i.onload=()=>{const k=Math.min(1,1600/Math.max(i.width,i.height)),c=document.createElement('canvas');c.width=i.width*k;c.height=i.height*k;c.getContext('2d').drawImage(i,0,0,c.width,c.height);c.toBlob(b=>b?ok(b):no(),'image/jpeg',.85)};i.onerror=no;i.src=URL.createObjectURL(f)})}
async function ln(){const f=$('#nf')&&$('#nf').files[0];if(!f){msg='Tire ou escolha a foto da nota primeiro';draw();return}
NT.busy=true;NT.err='';draw();
try{const sm=await claude.use('sample');if(!sm)throw 0;const b=await rz(f);
const r=await sm.json('Esta é a foto de uma nota de compra de mercadorias de uma conveniência. Liste cada item comprado. Responda SOMENTE com JSON no formato [{"codigo":"código de barras ou do produto, ou vazio","nome":"descrição","qtd":número,"custo":preço unitário em reais, número}]. Se só houver o total do item, divida pela quantidade. Não invente itens.',{images:[b],cache:false});
const L=Array.isArray(r)?r:(r&&r.itens)||[];
NT.it=L.map(x=>{const c=mt(x);return {c,n:String(x.nome||''),q:+x.qtd||0,cu:+x.custo||0,pv:c?S.prods[c].p:0,on:!!c}});rc();if(!NT.it.length)NT.err='Não encontrei itens na foto. Tente de mais perto e com boa luz.'}
catch(e){NT.err='Não consegui ler a nota. Tente outra foto, de perto e com boa luz.'}
NT.busy=false;draw()}
function ap(){let n=0;NT.it.forEach(x=>{if(!x.on||!x.c||!x.pv)return;const o=S.prods[x.c];S.prods[x.c]={n:o.n,p:x.pv,q:o.q+(NT.est?x.q:0),cu:x.cu};n++});NT.it=[];save();msg=n+' preço(s) atualizado(s) pela nota';draw()}
function nota(){const t=NT.it;
return `<div class=card><b>Nota de compra (foto)</b><div class=row><input id=nf type=file accept="image/*"><input type=number step=.1 placeholder="Margem % sobre o custo" value="${E(NT.mg)}" onchange="NT.mg=this.value;rc();if(NT.it.length)draw()"></div><div class=row><button class=g onclick="ln()" ${NT.busy?'disabled':''}>${NT.busy?'Lendo a nota...':'Ler nota'}</button></div>${NT.err?`<p>${NT.err}</p>`:''}
${t.length?`<table>${t.map((x,i)=>`<tr><td><input type=checkbox ${x.on?'checked':''} ${x.c?'':'disabled'} onchange="NT.it[${i}].on=this.checked"></td><td>${E(x.n)}<br><small>${x.c?x.q+' un · custo '+R(x.cu)+' · atual '+R(S.prods[x.c].p):'não encontrado no cadastro: cadastre em Produtos'}</small></td><td>${x.c?`<input type=number step=.01 value="${x.pv}" style="width:90px" onchange="NT.it[${i}].pv=+this.value">`:''}</td></tr>`).join('')}</table><label><input type=checkbox ${NT.est?'checked':''} onchange="NT.est=this.checked"> Dar entrada no estoque também</label><div class=row><button class=g onclick="ap()">Aplicar preços</button></div>`:''}</div>`}
function per(){const d=t=>new Date(t).toLocaleDateString('sv'),b=PR.b||d(Date.now()),a=PR.a||d(Date.now()-6*864e5),s=S.vendas.filter(v=>d(v.t)>=a&&d(v.t)<=b).reduce((x,v)=>x+v.v,0);
return `<div class=card><b>Faturamento por período</b><div class=row><input type=date value="${a}" onchange="PR.a=this.value;draw()"><input type=date value="${b}" onchange="PR.b=this.value;draw()"></div><div class=tot>${R(s)}</div></div>`}
</script></body></html>
