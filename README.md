# KATTAKURGON-SHIRINLIKLAR
import { useState, useMemo } from "react";
const DEFAULT_PRODUCTS = [
{ id: 1, name: "Нон (лепёшка)", unit: "дона", price: 1500, emoji: " { id: 2, name: "Самса", unit: "дона", price: 2000, emoji: " " },
{ id: 3, name: "Булочка", unit: "дона", price: 1200, emoji: " " },
{ id: 4, name: "Батон", unit: "дона", price: 3500, emoji: " " },
{ id: 5, name: "Слойка", unit: "дона", price: 2500, emoji: " " },
{ id: 6, name: "Пирожок", unit: "дона", price: 1800, emoji: " " },
{ id: 7, name: "Торт (кг)", unit: "кг", price: 50000, emoji: " " },
{ id: 8, name: "Печенье (кг)", unit: "кг", price: 25000, emoji: " " },
{ id: 9, name: "Лаваш", unit: "дона", price: 2000, emoji: " " },
{ id: 10, name: "Кекс", unit: "дона", price: 3000, emoji: " " },
" },
];
const DEFAULT_STORES = [
{ id: 1, name: "Дўкон №1", debt: 0 },
{ id: 2, name: "Дўкон №2", debt: 0 },
{ id: 3, name: "Дўкон №3", debt: 0 },
];
const STATUS = {
new: { label: "Янги", baking: { label: "Пиширилмоқда", color: "#fb923c", bg: "#2d1200", icon: " ready: { label: "Тайёр", delivered: { label: "Етказилди", color: "#60a5fa", bg: "#0f2040", icon: " " },
color: "#34d399", bg: "#0a2e1a", icon: " " },
color: "#9ca3af", bg: "#1a1a1a", icon: " " },
" },
};
const SF = ["new","baking","ready","delivered"];
function ld(k, fb) { try { const v = localStorage.getItem(k); return v ? JSON.parse(v) : fb;
function sv(k, v) { try { localStorage.setItem(k, JSON.stringify(v)); } catch {} }
let _nid = ld("nz_nid", 1000);
let _pid = ld("nz_pid", 2000);
function fmt(n) { return Number(n||0).toLocaleString("ru-RU"); }
function today() { return new Date().toISOString().split("T")[0]; }
function toDateStr(iso) { return iso ? iso.split("T")[0] : ""; }
export default function App() {
const [tab, setTab] = useState("orders");
const [orders, setOrders_] = useState(() => ld("nz_orders", []));
const [products, setProds_] = useState(() => ld("nz_prods", DEFAULT_PRODUCTS));
const [stores, setStores_] = useState(() => ld("nz_stores", DEFAULT_STORES));
const [payments, setPayments_] = useState(() => ld("nz_payments", [])); // {id,storeId,stor
const [detail, setDetail] = useState(null);
const [filterStatus, setFilterStatus] = useState("all");
const [filterDate, setFilterDate] = useState("");
// new order form
const [fStore, setFStore] = useState("");
const [fDate, setFDate] = useState("");
const [fItems, setFItems] = useState([]);
const [fNote, setFNote] = useState("");
const [fPaid, setFPaid] = useState(false); // оплачено сразу?
// catalog form
const [cName, setCName] = useState(""); const [cUnit, setCUnit] = useState("дона"); const [
// store form
const [sName, setSName] = useState("");
// payment form
const [payStore, setPayStore] = useState(""); const [payAmt, setPayAmt] = useState(""); con
// daily report state
const [repDate, setRepDate] = useState(today());
function setOrders(v) { setOrders_(v); sv("nz_orders", v); }
function setProds(v) { setProds_(v); sv("nz_prods", v); }
function setStores(v) { setStores_(v); sv("nz_stores", v); }
function setPayments(v) { setPayments_(v); sv("nz_payments", v); }
// ── item helpers ──
function addItem(pid) {
pid = Number(pid); if (!pid || fItems.find(i=>i.productId===pid)) return;
setFItems(p=>[...p,{productId:pid,qty:1}]);
}
function setQty(pid,v) { setFItems(p=>p.map(i=>i.productId===pid?{...i,qty:Math.max(1,Numbe
function rmItem(pid) { setFItems(p=>p.filter(i=>i.productId!==pid)); }
function calcTotal(items) {
return items.reduce((s,i)=>{
const p=products.find(p=>p.id===i.productId);
return s+(p?p.price*i.qty:0);
},0);
}
// ── submit order ──
function submitOrder() {
if (!fStore || fItems.length===0) return;
const total = calcTotal(fItems);
const o = {
id:++_nid, storeId:Number(fStore),
storeName: stores.find(s=>s.id===Number(fStore))?.name||"",
deliveryDate:fDate, items:fItems.map(i=>({...i})),
total, status:"new", paid:fPaid,
createdAt:new Date().toISOString(), note:fNote,
};
sv("nz_nid",_nid);
setOrders([o,...orders]);
// если не оплачено — добавить долг магазину
if (!fPaid) {
setStores(stores.map(s=> s.id===Number(fStore) ? {...s, debt:(s.debt||0)+total} : s));
}
setFStore(""); setFDate(""); setFItems([]); setFNote(""); setFPaid(false);
setTab("orders");
}
// ── advance status ──
function advance(oid) {
setOrders(orders.map(o=>{
if(o.id!==oid) return o;
const idx=SF.indexOf(o.status);
return {...o, status:SF[Math.min(idx+1,3)]};
}));
}
// ── mark paid ──
function markPaid(oid) {
const o = orders.find(x=>x.id===oid);
if (!o || o.paid) return;
setOrders(orders.map(x=>x.id===oid?{...x,paid:true}:x));
setStores(stores.map(s=>s.id===o.storeId?{...s,debt:Math.max(0,(s.debt||0)-o.total)}:s));
}
// ── delete order ──
function delOrder(oid) {
const o = orders.find(x=>x.id===oid);
if (!o) return;
setOrders(orders.filter(x=>x.id!==oid));
// агар тўланмаган бўлса — қарзни камайтир
if (!o.paid) {
setStores(stores.map(s=>s.id===o.storeId?{...s,debt:Math.max(0,(s.debt||0)-o.total)}:s)
}
setDetail(null);
}
// ── add payment ──
function addPayment() {
if (!payStore || !payAmt || Number(payAmt)<=0) return;
const sid = Number(payStore);
const amt = Number(payAmt);
const p = {
id:++_pid, storeId:sid,
storeName:stores.find(s=>s.id===sid)?.name||"",
amount:amt, date:today(), note:payNote,
};
sv("nz_nid",_nid);
setPayments([p,...payments]);
setStores(stores.map(s=>s.id===sid?{...s,debt:Math.max(0,(s.debt||0)-amt)}:s));
setPayStore(""); setPayAmt(""); setPayNote("");
}
// ── daily report ──
const report = useMemo(()=>{
const dayOrders = orders.filter(o=> toDateStr(o.createdAt)===repDate);
const revenue = dayOrders.reduce((s,o)=>s+o.total,0);
const paid = dayOrders.filter(o=>o.paid).reduce((s,o)=>s+o.total,0);
const debt = dayOrders.filter(o=>!o.paid).reduce((s,o)=>s+o.total,0);
const delivered = dayOrders.filter(o=>o.status==="delivered").length;
// product breakdown
const prodMap = {};
dayOrders.forEach(o=>o.items.forEach(i=>{
const p=products.find(p=>p.id===i.productId);
if(!p) return;
if(!prodMap[p.id]) prodMap[p.id]={name:p.name,emoji:p.emoji,unit:p.unit,qty:0,sum:0};
prodMap[p.id].qty += i.qty;
prodMap[p.id].sum += i.qty*p.price;
}));
// store breakdown
const storeMap = {};
dayOrders.forEach(o=>{
if(!storeMap[o.storeId]) storeMap[o.storeId]={name:o.storeName,total:0,count:0};
storeMap[o.storeId].total+=o.total;
storeMap[o.storeId].count++;
});
return { dayOrders, revenue, paid, debt, delivered, prodMap, storeMap };
},[orders, repDate, products]);
const totalDebt = stores.reduce((s,st)=>s+(st.debt||0),0);
const filtered = (filterStatus==="all"?orders:orders.filter(o=>o.status===filterStatus))
.filter(o=>!filterDate||toDateStr(o.createdAt)===filterDate);
const counts = Object.fromEntries(SF.map(s=>[s,orders.filter(o=>o.status===s).length]));
// ════════════════════════════════════════════════════════════════
// DETAIL VIEW
// ════════════════════════════════════════════════════════════════
if (detail!==null) {
const o=orders.find(x=>x.id===detail);
if(!o){setDetail(null);return null;}
const sc=STATUS[o.status];
const nextS=SF[SF.indexOf(o.status)+1];
return (
<div style={S.page}>
<div style={S.wrap}>
<button onClick={()=>setDetail(null)} style={S.back}>← Орқага</button>
<div style={{display:"flex",alignItems:"center",gap:14,padding:"14px 0 18px"}}>
<div style={{width:50,height:50,borderRadius:14,background:sc.bg,border:`1.5px so
<div style={{flex:1}}>
<div style={{fontSize:11,color:"#555",letterSpacing:1}}>БУЮРТМА #{o.id}</div>
<div style={{fontSize:20,fontWeight:900,color:"#f5f0e8",marginTop:2}}>{o.storeN
</div>
<div style={{background:sc.bg,color:sc.color,border:`1px solid ${sc.color}44`,bor
</div>
<div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:12}}>
{[
[" Магазин", o.storeName],
[" Доставка", o.deliveryDate||"Кўрсатилмаган"],
[" Сана", new Date(o.createdAt).toLocaleString("ru-RU",{day:"2-digit",m
[" Сумма", fmt(o.total)+" сум"],
].map(([k,v])=>(
<div key={k} style={{background:"#13131d",border:"1px solid #252535",borderRadi
<div style={{fontSize:11,color:"#555",marginBottom:3}}>{k}</div>
<div style={{fontSize:13,fontWeight:700,color:"#f5f0e8"}}>{v}</div>
</div>
))}
</div>
{/* paid badge */}
<div style={{marginBottom:12}}>
{o.paid
? <span style={{background:"#0a2e1a",color:"#34d399",border:"1px solid #34d3994
: <span style={{background:"#2e1a0a",color:"#fb923c",border:"1px solid #fb923c4
}
</div>
{/* items */}
<div style={{background:"#13131d",border:"1px solid #252535",borderRadius:14,overfl
<div style={{padding:"10px 14px",borderBottom:"1px solid #1e1e2e",fontSize:11,col
{o.items.map((item,i)=>{
const p=products.find(p=>p.id===item.productId);
return (
<div key={i} style={{display:"flex",alignItems:"center",gap:10,padding:"11px
<div style={{fontSize:22}}>{p?.emoji||" "}</div>
<div style={{flex:1}}>
<div style={{fontSize:14,fontWeight:600,color:"#f5f0e8"}}>{p?.name||"—"}<
<div style={{fontSize:12,color:"#666"}}>{item.qty} {p?.unit} × {fmt(p?.pr
</div>
<div style={{fontWeight:800,color:"#fbbf24"}}>{fmt((p?.price||0)*item.qty)}
</div>
);
})}
<div style={{display:"flex",justifyContent:"space-between",padding:"12px 16px",ba
<span style={{color:"#888",fontWeight:700}}>ЖАМИ</span>
<span style={{fontSize:16,fontWeight:900,color:"#34d399"}}>{fmt(o.total)} сум</
</div>
</div>
{o.note&&<div style={{background:"#13131d",border:"1px solid #252535",borderRadius:
<div style={{fontSize:11,color:"#555",marginBottom:3}}> ИЗОҲ</div>
<div style={{fontSize:13,color:"#aaa"}}>{o.note}</div>
</div>}
{nextS&&<button onClick={()=>{advance(o.id);setDetail(null);}} style={{...S.btn,bac
→ {STATUS[nextS].label}га ўтказиш
</button>}
{!o.paid&&<button onClick={()=>{markPaid(o.id);setDetail(null);}} style={{...S.btn,
Тўланди деб белгилаш
</button>}
<button onClick={()=>{if(window.confirm("Ўчириш?"))delOrder(o.id);}} style={S.dange
</div>
</div>
);
}
// ════════════════════════════════════════════════════════════════
// MAIN
// ════════════════════════════════════════════════════════════════
return (
<div style={S.page}>
<div style={S.wrap}>
{/* HEADER */}
<div style={{padding:"22px 0 0"}}>
<div style={{fontSize:10,color:"#3a3a5a",letterSpacing:2.5,textTransform:"uppercase
<div style={{fontSize:24,fontWeight:900,color:"#f5f0e8",letterSpacing:-0.5,marginTo
{tab==="orders"&&" Буюртмалар"}
{tab==="new"&&" Янги буюртма"}
{tab==="products"&&" Маҳсулотлар"}
{tab==="stores"&&" Магазинлар"}
{tab==="report"&&" Кунлик ҳисоб"}
</div>
</div>
{/* ── ORDERS ─────────────────────────────────────── */}
{tab==="orders"&&(
<div style={{paddingTop:14}}>
{/* filter row */}
<div style={{display:"flex",gap:8,overflowX:"auto",paddingBottom:4,marginBottom:1
{[["all","Барчаси",orders.length],...SF.map(s=>[s,STATUS[s].label,counts[s]])].
const active=filterStatus===v;
const c=v==="all"?"#a78bfa":STATUS[v]?.color;
return <button key={v} onClick={()=>setFilterStatus(v)} style={{padding:"5px
})}
</div>
<input type="date" value={filterDate} onChange={e=>setFilterDate(e.target.value)}
{filtered.length===0
?<div style={{textAlign:"center",padding:"60px 0",color:"#333"}}><div style={{f
:filtered.map(o=>{
const sc=STATUS[o.status]; const nextS=SF[SF.indexOf(o.status)+1];
return (
<div key={o.id} onClick={()=>setDetail(o.id)} style={{background:"#13131d",
onMouseEnter={e=>e.currentTarget.style.borderColor=sc.color+"55"}
onMouseLeave={e=>e.currentTarget.style.borderColor="#252535"}>
<div style={{display:"flex",justifyContent:"space-between",alignItems:"fl
<div>
<div style={{fontSize:10,color:"#444"}}>#{o.id} · {toDateStr(o.create
<div style={{fontSize:16,fontWeight:800,color:"#f5f0e8",marginTop:2}}
{o.deliveryDate&&<div style={{fontSize:11,color:"#666",marginTop:2}}>
</div>
<div style={{textAlign:"right"}}>
<div style={{background:sc.bg,color:sc.color,border:`1px solid ${sc.c
<div style={{fontSize:15,fontWeight:900,color:"#34d399",marginTop:5}}
{!o.paid&&<div style={{fontSize:10,color:"#fb923c",marginTop:2}}> Қ
</div>
</div>
<div style={{display:"flex",flexWrap:"wrap",gap:5,marginBottom:nextS?8:0}
{o.items.map((item,i)=>{
const p=products.find(p=>p.id===item.productId);
return <span key={i} style={{fontSize:11,background:"#1e1e2e",color:"
})}
</div>
{nextS&&<button onClick={e=>{e.stopPropagation();advance(o.id);}} style={
</div>
);
})
}
</div>
)}
{/* ── NEW ORDER ───────────────────────────────────── */}
{tab==="new"&&(
<div style={{paddingTop:16}}>
<Lbl> Магазин *</Lbl>
<select value={fStore} onChange={e=>setFStore(e.target.value)} style={S.input}>
<option value="">— Танланг —</option>
{stores.map(s=><option key={s.id} value={s.id}>{s.name}{(s.debt||0)>0?` (қарз:
</select>
<Lbl> Етказиш санаси</Lbl>
<input type="date" value={fDate} min={today()} onChange={e=>setFDate(e.target.val
<Lbl> Маҳсулот қўшиш</Lbl>
<select onChange={e=>{addItem(e.target.value);e.target.value="";}} style={S.input
<option value="">— Маҳсулотни танланг —</option>
{products.filter(p=>!fItems.find(i=>i.productId===p.id)).map(p=>(
<option key={p.id} value={p.id}>{p.emoji} {p.name} — {fmt(p.price)} сум/{p.un
))}
</select>
{fItems.length>0&&(
<div style={{background:"#13131d",border:"1px solid #252535",borderRadius:14,ov
{fItems.map((item,i)=>{
const p=products.find(p=>p.id===item.productId);
return (
<div key={item.productId} style={{display:"flex",alignItems:"center",gap:
<div style={{fontSize:20}}>{p?.emoji}</div>
<div style={{flex:1}}>
<div style={{fontSize:13,fontWeight:600,color:"#f5f0e8"}}>{p?.name}</
<div style={{fontSize:11,color:"#666"}}>{fmt(p?.price)} × {item.qty}
</div>
<input type="number" value={item.qty} min={1} onChange={e=>setQty(item.
<button onClick={()=>rmItem(item.productId)} style={{background:"none",
</div>
);
})}
<div style={{display:"flex",justifyContent:"space-between",padding:"11px 14px
<span style={{fontSize:13,color:"#888"}}>ЖАМИ</span>
<span style={{fontSize:16,fontWeight:900,color:"#34d399"}}>{fmt(calcTotal(f
</div>
</div>
)}
{/* paid toggle */}
<div style={{display:"flex",alignItems:"center",gap:12,margin:"12px 0",padding:"1
<div style={{width:42,height:24,borderRadius:12,background:fPaid?"#065f46":"#25
<div style={{position:"absolute",top:2,left:fPaid?18:2,width:16,height:16,bor
</div>
<div>
<div style={{fontSize:13,fontWeight:600,color:"#f5f0e8"}}>Нақд тўланди</div>
<div style={{fontSize:11,color:"#666"}}>{fPaid?"Тўланган — қарзга кирмайди":"
</div>
</div>
<Lbl> Изоҳ</Lbl>
<textarea value={fNote} onChange={e=>setFNote(e.target.value)} placeholder="Қўшим
<button onClick={submitOrder} disabled={!fStore||fItems.length===0} style={{...S.
Буюртмани сақлаш
</button>
</div>
)}
{/* ── STORES (with debt) ──────────────────────────── */}
{tab==="stores"&&(
<div style={{paddingTop:16}}>
{/* total debt banner */}
{totalDebt>0&&(
<div style={{background:"#2d1200",border:"1px solid #fb923c44",borderRadius:14,
<div>
<div style={{fontSize:11,color:"#fb923c",letterSpacing:1}}>УМУМИЙ ҚАРЗ</div
<div style={{fontSize:22,fontWeight:900,color:"#fb923c",marginTop:2}}>{fmt(
</div>
<div style={{fontSize:32}}> </div>
</div>
)}
{stores.map(s=>(
<div key={s.id} style={{background:"#13131d",border:`1px solid ${(s.debt||0)>0?
<div style={{display:"flex",alignItems:"center",gap:10}}>
<div style={{fontSize:22}}> </div>
<div style={{flex:1}}>
<div style={{fontSize:15,fontWeight:700,color:"#f5f0e8"}}>{s.name}</div>
{(s.debt||0)>0
?<div style={{fontSize:12,color:"#fb923c",marginTop:2,fontWeight:700}}>
:<div style={{fontSize:12,color:"#34d399",marginTop:2}}> Қарзи йўқ</d
}
</div>
<button onClick={()=>setStores(stores.filter(x=>x.id!==s.id))} style={{back
</div>
</div>
))}
{/* payment form */}
<div style={{background:"#13131d",border:"1px solid #252535",borderRadius:14,padd
<div style={{fontSize:12,color:"#34d399",letterSpacing:1,marginBottom:12,fontWe
<Lbl>Магазин</Lbl>
<select value={payStore} onChange={e=>setPayStore(e.target.value)} style={S.inp
<option value="">— Танланг —</option>
{stores.filter(s=>(s.debt||0)>0).map(s=><option key={s.id} value={s.id}>{s.na
</select>
<Lbl>Сумма (сум)</Lbl>
<input type="number" value={payAmt} onChange={e=>setPayAmt(e.target.value)} pla
<Lbl>Изоҳ (ихтиёрий)</Lbl>
<input value={payNote} onChange={e=>setPayNote(e.target.value)} placeholder="На
<button onClick={addPayment} disabled={!payStore||!payAmt} style={{...S.btn,bac
Тўловни қайд этиш
</button>
</div>
{/* payment history */}
{payments.length>0&&(
<div>
<div style={{fontSize:12,color:"#555",letterSpacing:1,marginBottom:8}}>ТЎЛОВЛ
{payments.slice(0,10).map(p=>(
<div key={p.id} style={{background:"#13131d",border:"1px solid #0a2e1a",bor
<div>
<div style={{fontSize:13,fontWeight:600,color:"#f5f0e8"}}>{p.storeName}
<div style={{fontSize:11,color:"#555"}}>{p.date}{p.note?` · ${p.note}`:
</div>
<div style={{fontSize:14,fontWeight:800,color:"#34d399"}}>+{fmt(p.amount)
</div>
))}
</div>
)}
{/* add store */}
<div style={{background:"#13131d",border:"1px solid #252535",borderRadius:14,padd
<div style={{fontSize:12,color:"#555",letterSpacing:1,marginBottom:12}}>ЯНГИ МА
<Lbl>Номи</Lbl>
<input value={sName} onChange={e=>setSName(e.target.value)} placeholder="Дўкон
<button onClick={()=>{if(!sName.trim())return;setStores([...stores,{id:Date.now
</div>
</div>
)}
{/* ── PRODUCTS ───────────────────────────────────── */}
{tab==="products"&&(
<div style={{paddingTop:16}}>
{products.map(p=>(
<div key={p.id} style={{...S.row,marginBottom:8}}>
<div style={{fontSize:22,marginRight:11}}>{p.emoji}</div>
<div style={{flex:1}}>
<div style={{fontSize:14,fontWeight:700,color:"#f5f0e8"}}>{p.name}</div>
<div style={{fontSize:12,color:"#666",marginTop:1}}>{fmt(p.price)} сум / {p
</div>
<button onClick={()=>setProds(products.filter(x=>x.id!==p.id))} style={{backg
</div>
))}
<div style={{background:"#13131d",border:"1px solid #252535",borderRadius:14,padd
<div style={{fontSize:12,color:"#555",letterSpacing:1,marginBottom:12}}>ЯНГИ МА
<Lbl>Номи</Lbl>
<input value={cName} onChange={e=>setCName(e.target.value)} placeholder="Самса,
<div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10}}>
<div><Lbl>Бирлик</Lbl><select value={cUnit} onChange={e=>setCUnit(e.target.va
<div><Lbl>Нарх (сум)</Lbl><input type="number" value={cPrice} onChange={e=>se
</div>
<button onClick={()=>{
if(!cName.trim()||!cPrice)return;
const em={"дона":" ","кг":" ","г":" ","упак":" ","л":" "};
setProds([...products,{id:Date.now(),name:cName.trim(),unit:cUnit,price:Numbe
setCName("");setCPrice("");
}} style={S.btn}>+ Қўшиш</button>
</div>
</div>
)}
{/* ── DAILY REPORT ────────────────────────────────── */}
{tab==="report"&&(
<div style={{paddingTop:16,paddingBottom:20}}>
<Lbl> Сана</Lbl>
<input type="date" value={repDate} onChange={e=>setRepDate(e.target.value)} style
{/* summary cards */}
<div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:16}
{[
[" Буюртмалар", report.dayOrders.length+" та","#60a5fa"],
[" Умумий", fmt(report.revenue)+" сум","#fbbf24"],
[" Тўланган", fmt(report.paid)+" сум","#34d399"],
[" Қарз", fmt(report.debt)+" сум","#fb923c"],
].map(([k,v,c])=>(
<div key={k} style={{background:"#13131d",border:`1px solid ${c}22`,borderRad
<div style={{fontSize:11,color:"#555",marginBottom:4}}>{k}</div>
<div style={{fontSize:16,fontWeight:900,color:c}}>{v}</div>
</div>
))}
</div>
{/* product breakdown */}
{Object.keys(report.prodMap).length>0&&(
<div style={{background:"#13131d",border:"1px solid #252535",borderRadius:14,ov
<div style={{padding:"10px 14px",borderBottom:"1px solid #1e1e2e",fontSize:11
{Object.values(report.prodMap).sort((a,b)=>b.sum-a.sum).map((p,i,arr)=>(
<div key={p.name} style={{display:"flex",alignItems:"center",padding:"11px
<div style={{fontSize:20}}>{p.emoji}</div>
<div style={{flex:1}}>
<div style={{fontSize:13,fontWeight:600,color:"#f5f0e8"}}>{p.name}</div
<div style={{fontSize:11,color:"#666"}}>{p.qty} {p.unit}</div>
</div>
<div style={{fontWeight:800,color:"#fbbf24",fontSize:14}}>{fmt(p.sum)} су
</div>
))}
</div>
)}
{/* store breakdown */}
{Object.keys(report.storeMap).length>0&&(
<div style={{background:"#13131d",border:"1px solid #252535",borderRadius:14,ov
<div style={{padding:"10px 14px",borderBottom:"1px solid #1e1e2e",fontSize:11
{Object.values(report.storeMap).sort((a,b)=>b.total-a.total).map((s,i,arr)=>(
<div key={s.name} style={{display:"flex",justifyContent:"space-between",ali
<div>
<div style={{fontSize:13,fontWeight:600,color:"#f5f0e8"}}>{s.name}</div
<div style={{fontSize:11,color:"#666"}}>{s.count} та буюртма</div>
</div>
<div style={{fontWeight:800,color:"#34d399"}}>{fmt(s.total)} сум</div>
</div>
))}
</div>
)}
{report.dayOrders.length===0&&(
<div style={{textAlign:"center",padding:"50px 0",color:"#333"}}>
<div style={{fontSize:48}}> </div>
<div style={{marginTop:10}}>Бу кунда буюртма йўқ</div>
</div>
)}
</div>
)}
</div>
{/* BOTTOM NAV */}
<nav style={{position:"fixed",bottom:0,left:0,right:0,background:"#0a0a10",borderTop:"1
{[
["orders"," ","Буюртма"],
["new"," ","Янги"],
["report"," ","Ҳисоб"],
["stores"," ","Магазин"],
["products"," ","Маҳсулот"],
].map(([key,icon,label])=>(
<button key={key} onClick={()=>setTab(key)} style={{flex:1,padding:"10px 0 12px",ba
<span style={{fontSize:18}}>{icon}</span>
<span style={{fontSize:9,letterSpacing:0.2,fontWeight:tab===key?700:400}}>{label}
{key==="stores"&&totalDebt>0&&<span style={{width:6,height:6,borderRadius:3,backg
</button>
))}
</nav>
</div>
);
}
function Lbl({children}) {
return <div style={{fontSize:11,color:"#555",letterSpacing:1,textTransform:"uppercase",marg
}
const S = {
page:{minHeight:"100vh",background:"#0a0a10",color:"#f5f0e8",fontFamily:"'Georgia','Palatin
wrap:{maxWidth:480,margin:"0 auto",padding:"0 16px 90px"},
input:{width:"100%",padding:"11px 12px",background:"#13131d",border:"1px solid #252535",bor
btn:{width:"100%",marginTop:12,padding:"13px",background:"linear-gradient(135deg,#b45309,#f
dangerBtn:{width:"100%",padding:"11px",background:"#200a0a",border:"1px solid #ef444433",bo
back:{background:"none",border:"none",color:"#555",fontSize:13,cursor:"pointer",fontFamily:
row:{background:"#13131d",border:"1px solid #252535",borderRadius:12,padding:"11px 13px",di
};