import { useState, useEffect } from "react";

// ─── Constants ────────────────────────────────────────────────────────────────
const DEFAULT_STUDENTS = ["Derrick", "Tom", "Mike", "Jordan", "Lael", "Philip", "Sam"];
const COACH_PIN = "1234";
const WEEK_CONFIG = [
  { week:1, label:"Foundation",         contracts:1, dailyGoal:100,  weeklyTarget:500,  cumulTarget:500  },
  { week:2, label:"Controlled Scaling", contracts:3, dailyGoal:500,  weeklyTarget:2500, cumulTarget:3000 },
  { week:3, label:"Full Execution",     contracts:5, dailyGoal:1000, weeklyTarget:5000, cumulTarget:8000 },
];
const DAY_NAMES = ["Monday","Tuesday","Wednesday","Thursday","Friday"];
const CHALLENGE_GOAL = 8000;
const C = {
  gold:"#D4A017", dark:"#0D0D1A", mid:"#12121F", card:"#1A1A2E",
  accent:"#0F3460", green:"#00C853", red:"#FF1744", amber:"#FFB300",
  week1:"#1B5E20", week2:"#0D47A1", week3:"#6A1E6A", purple:"#9C27B0",
};

// ─── Helpers ──────────────────────────────────────────────────────────────────
const fmt = v => (v==null||v==="") ? "—" : `$${Number(v).toLocaleString("en-US",{minimumFractionDigits:2,maximumFractionDigits:2})}`;
const emptyTrade   = () => ({ entryTime:"", callPut:"", contracts:"", entryPrice:"", exitPrice:"", rulesMet:"", screenshot:"", note:"" });
const emptyDay     = () => ({ trades:[emptyTrade(),emptyTrade(),emptyTrade()], spread:{type:"",timeIn:"",timeOut:"",strikes:"",creditRecvd:"",maxRisk:"",contracts:"",outcome:"",notes:""}, flatScreenshot:"" });
const emptyStudent = () => ({ weeks:[1,2,3].map(w=>({ week:w, days:DAY_NAMES.map(()=>emptyDay()) })) });

// ─── Calculations ─────────────────────────────────────────────────────────────
const calcGross = t => {
  if (!t.entryPrice||!t.exitPrice||!t.contracts) return null;
  return Math.round((parseFloat(t.exitPrice)-parseFloat(t.entryPrice))*parseFloat(t.contracts)*100*100)/100;
};
const calcDayNet = day => {
  let tot=0, has=false;
  day.trades.forEach(t=>{ const g=calcGross(t); if(g!==null){tot+=g;has=true;} });
  const sp=parseFloat(day.spread?.outcome)||0; tot+=sp;
  return (has||sp) ? tot : null;
};
const calcWeekTotal      = wk => wk.days.reduce((s,d)=>s+(calcDayNet(d)||0),0);
const calcChallengeTotal = sd => sd.weeks.reduce((s,w)=>s+calcWeekTotal(w),0);
const countLosses        = day => day.trades.filter(t=>{ const g=calcGross(t); return g!==null&&g<0; }).length;

// ─── Storage ──────────────────────────────────────────────────────────────────
async function loadRoster(){
  try{ const r=await window.storage.get("roster",true); return r?JSON.parse(r.value):DEFAULT_STUDENTS; }
  catch{ return DEFAULT_STUDENTS; }
}
async function saveRoster(list){
  try{ await window.storage.set("roster",JSON.stringify(list),true); }catch(e){console.error(e);}
}
async function loadAll(roster){
  const res={};
  for(const s of roster){
    try{ const r=await window.storage.get(`student:${s}`,true); res[s]=r?JSON.parse(r.value):emptyStudent(); }
    catch{ res[s]=emptyStudent(); }
  }
  return res;
}
async function saveStudent(name,data){
  try{ await window.storage.set(`student:${name}`,JSON.stringify(data),true); }catch(e){console.error(e);}
}

// ─── Style atoms ──────────────────────────────────────────────────────────────
const TD    = { padding:"6px 8px", verticalAlign:"middle" };
const TH    = { padding:"8px 8px", fontSize:10, fontWeight:700, color:"#555", letterSpacing:1, textTransform:"uppercase", textAlign:"left", borderBottom:"1px solid #1e1e2e", whiteSpace:"nowrap" };
const IBASE = { background:"#0d0d1a", border:"1px solid #2a2a3e", borderRadius:6, padding:"6px 8px", color:"#e0e0e0", fontSize:12, fontFamily:"'DM Mono',monospace", outline:"none", width:"100%" };
const SINP  = { ...IBASE, width:92, border:"1px solid #4a0e6a" };

// ─── Shared UI ────────────────────────────────────────────────────────────────
function ProgressBar({ value, max, color }){
  const pct=Math.min(100,Math.max(0,(value/max)*100));
  return <div style={{background:"#1a1a2e",borderRadius:6,height:8,overflow:"hidden",margin:"4px 0"}}>
    <div style={{width:`${pct}%`,height:"100%",background:`linear-gradient(90deg,${color}88,${color})`,borderRadius:6,transition:"width .6s cubic-bezier(.4,0,.2,1)"}}/>
  </div>;
}
function StatBox({label,value,color,sub}){
  return <div style={{background:"#12121f",border:`1px solid ${color}33`,borderRadius:10,padding:"14px 18px",minWidth:110}}>
    <div style={{fontSize:10,color:"#555",letterSpacing:1,textTransform:"uppercase",marginBottom:4}}>{label}</div>
    <div style={{fontSize:20,fontWeight:800,color,fontFamily:"'DM Mono',monospace"}}>{value}</div>
    {sub&&<div style={{fontSize:10,color:"#555",marginTop:2}}>{sub}</div>}
  </div>;
}
function Badge({text,color,bg}){
  return <span style={{display:"inline-block",padding:"2px 10px",borderRadius:20,fontSize:11,fontWeight:700,color:color||"#fff",background:bg||"#ffffff22",letterSpacing:.5,whiteSpace:"nowrap"}}>{text}</span>;
}

// ─── Trade Row ────────────────────────────────────────────────────────────────
function TradeRow({ trade, idx, onChange, onRemove, canRemove, locked }){
  const gross=calcGross(trade);
  const isLoss=gross!==null&&gross<0;
  const isWin =gross!==null&&gross>0;
  const inp=(f,ph,type="text")=>(
    <input type={type} placeholder={ph} value={trade[f]} disabled={locked}
      onChange={e=>onChange({...trade,[f]:e.target.value})} style={{...IBASE,opacity:locked?.5:1}}/>
  );
  const sel=(f,opts)=>(
    <select value={trade[f]} disabled={locked} onChange={e=>onChange({...trade,[f]:e.target.value})}
      style={{...IBASE,opacity:locked?.5:1}}>
      {opts.map(o=><option key={o.v} value={o.v}>{o.l}</option>)}
    </select>
  );
  return (
    <tr style={{background:isLoss?"#1a0808":isWin?"#081a08":idx%2===0?"#12121f":"#0f0f1c",borderLeft:`3px solid ${isLoss?C.red:isWin?C.green:"#2a2a3e"}`}}>
      <td style={{...TD,width:32}}>
        <div style={{display:"flex",alignItems:"center",gap:3}}>
          <span style={{color:"#444",fontSize:10,minWidth:16}}>T{idx+1}</span>
          {canRemove&&!locked&&(
            <button onClick={onRemove} title="Remove row"
              style={{background:"none",border:"none",cursor:"pointer",color:"#ff444466",fontSize:15,padding:0,lineHeight:1}}
              onMouseEnter={e=>e.target.style.color=C.red}
              onMouseLeave={e=>e.target.style.color="#ff444466"}>×</button>
          )}
        </div>
      </td>
      <td style={TD}>{inp("entryTime","9:32 AM")}</td>
      <td style={TD}>{sel("callPut",[{v:"",l:"—"},{v:"CALL",l:"📈 CALL"},{v:"PUT",l:"📉 PUT"}])}</td>
      <td style={TD}>{inp("contracts","1","number")}</td>
      <td style={TD}>{inp("entryPrice","0.00","number")}</td>
      <td style={TD}>{inp("exitPrice","0.00","number")}</td>
      <td style={{...TD,fontFamily:"'DM Mono',monospace",fontWeight:700,color:isLoss?C.red:isWin?C.green:"#555",whiteSpace:"nowrap"}}>
        {gross!==null?fmt(gross):"—"}
      </td>
      <td style={TD}>{sel("rulesMet",[{v:"",l:"—"},{v:"YES",l:"✅ Yes"},{v:"PARTIAL",l:"⚠️ Partial"},{v:"NO",l:"❌ No"}])}</td>
      <td style={TD}>{inp("screenshot","img_001.png")}</td>
      <td style={TD}>{inp("note","optional note")}</td>
    </tr>
  );
}

// ─── Day Block ────────────────────────────────────────────────────────────────
function DayBlock({ dayIdx, day, onChange, weekConfig, locked }){
  const dayNet   =calcDayNet(day);
  const losses   =countLosses(day);
  const threeLoss=losses>=3;
  const hasTrades=day.trades.some(t=>calcGross(t)!==null);
  const isFlat   =!hasTrades;
  const goalHit  =dayNet!==null&&dayNet>=weekConfig.dailyGoal;
  const isLossDay=dayNet!==null&&dayNet<0;
  const spreadOut=parseFloat(day.spread?.outcome)||0;

  const updateTrade=(i,t)=>onChange({...day,trades:day.trades.map((tr,idx)=>idx===i?t:tr)});
  const addTrade   =()=>onChange({...day,trades:[...day.trades,emptyTrade()]});
  const removeTrade=i=>{ if(day.trades.length<=1)return; onChange({...day,trades:day.trades.filter((_,idx)=>idx!==i)}); };
  const updSpread  =(f,v)=>onChange({...day,spread:{...day.spread,[f]:v}});

  const statusColor=goalHit?C.green:isLossDay?C.red:C.amber;
  const statusText =goalHit?"✅ GOAL HIT":isLossDay?"❌ LOSS DAY":hasTrades?"⚠️ IN PROGRESS":"—";

  return (
    <div style={{marginBottom:16,borderRadius:12,overflow:"hidden",border:`1px solid ${goalHit?C.green+"44":isLossDay?C.red+"44":"#1e1e2e"}`}}>
      {/* Header */}
      <div style={{background:"linear-gradient(135deg,#1a1a2e,#12121f)",padding:"10px 16px",display:"flex",alignItems:"center",justifyContent:"space-between",borderBottom:"1px solid #1e1e2e",flexWrap:"wrap",gap:8}}>
        <div style={{display:"flex",alignItems:"center",gap:10,flexWrap:"wrap"}}>
          <span style={{fontWeight:800,fontSize:13,color:"#ccc"}}>DAY {dayIdx+1} — {DAY_NAMES[dayIdx]}</span>
          {threeLoss&&<Badge text="🛑 3 LOSSES — STOP TRADING TODAY" bg="#cc000033" color={C.red}/>}
          {isFlat&&<Badge text="📸 FLAT — SCREENSHOT REQUIRED" bg="#D4A01722" color={C.gold}/>}
        </div>
        <div style={{display:"flex",alignItems:"center",gap:12}}>
          <span style={{fontFamily:"'DM Mono',monospace",fontWeight:800,fontSize:16,color:statusColor}}>{dayNet!==null?fmt(dayNet):"—"}</span>
          <Badge text={statusText} bg={statusColor+"22"} color={statusColor}/>
        </div>
      </div>

      {/* Trades table */}
      <div style={{overflowX:"auto"}}>
        <table style={{width:"100%",borderCollapse:"collapse"}}>
          <thead>
            <tr style={{background:"#0d0d1a"}}>
              {["#","ENTRY TIME","CALL/PUT","CONTRACTS","ENTRY $","EXIT $","GROSS P&L","RULES MET?","SCREENSHOT","NOTE"].map(h=>(
                <th key={h} style={TH}>{h}</th>
              ))}
            </tr>
          </thead>
          <tbody>
            {day.trades.map((t,i)=>(
              <TradeRow key={i} trade={t} idx={i}
                onChange={nt=>updateTrade(i,nt)}
                onRemove={()=>removeTrade(i)}
                canRemove={day.trades.length>1}
                locked={locked}/>
            ))}
          </tbody>
        </table>
      </div>

      {/* Add row */}
      {!locked&&(
        <div style={{padding:"8px 16px",background:"#0a0a14",borderTop:"1px solid #1e1e2e",display:"flex",alignItems:"center",justifyContent:"space-between",flexWrap:"wrap",gap:8}}>
          <button onClick={addTrade}
            style={{display:"flex",alignItems:"center",gap:7,background:"none",border:`1px dashed ${C.gold}55`,borderRadius:8,padding:"7px 16px",color:C.gold,fontSize:12,fontWeight:700,cursor:"pointer",transition:"all .15s"}}
            onMouseEnter={e=>{e.currentTarget.style.background=C.gold+"11";e.currentTarget.style.borderColor=C.gold;}}
            onMouseLeave={e=>{e.currentTarget.style.background="none";e.currentTarget.style.borderColor=C.gold+"55";}}>
            <span style={{fontSize:20,lineHeight:1,marginTop:-2}}>＋</span> Add Trade Row
          </button>
          <span style={{fontSize:10,color:"#333",fontStyle:"italic"}}>Scaling out? Add a row per exit — same entry, partial contracts, each exit price.</span>
        </div>
      )}

      {/* Flat screenshot */}
      {isFlat&&(
        <div style={{padding:"8px 16px",background:"#0f0f1c",borderTop:"1px solid #1e1e2e",display:"flex",alignItems:"center",gap:10,flexWrap:"wrap"}}>
          <span style={{fontSize:11,color:C.gold,fontWeight:700,whiteSpace:"nowrap"}}>📸 FLAT DAY — Screenshot Ref:</span>
          <input type="text" placeholder="e.g. flat_monday.png" value={day.flatScreenshot||""} disabled={locked}
            onChange={e=>onChange({...day,flatScreenshot:e.target.value})}
            style={{flex:1,minWidth:140,...IBASE,border:"1px solid #D4A01744"}}/>
        </div>
      )}

      {/* Subtotals bar */}
      {hasTrades&&(
        <div style={{padding:"6px 16px",background:"#0d0d18",borderTop:"1px solid #1e1e2e",display:"flex",gap:14,flexWrap:"wrap",alignItems:"center"}}>
          <span style={{fontSize:10,color:"#333",letterSpacing:1,flexShrink:0}}>BREAKDOWN:</span>
          {day.trades.map((t,i)=>{ const g=calcGross(t); if(g===null)return null;
            return <span key={i} style={{fontSize:11,fontFamily:"'DM Mono',monospace",fontWeight:700,color:g<0?C.red:C.green}}>T{i+1}: {fmt(g)}</span>;
          })}
          {spreadOut!==0&&<span style={{fontSize:11,fontFamily:"'DM Mono',monospace",fontWeight:700,color:spreadOut>0?C.green:C.red}}>SPREAD: {fmt(spreadOut)}</span>}
          <span style={{marginLeft:"auto",fontSize:12,fontFamily:"'DM Mono',monospace",fontWeight:800,color:statusColor,flexShrink:0}}>NET: {fmt(dayNet)}</span>
        </div>
      )}

      {/* Credit spread */}
      <div style={{background:"#0d0d14",borderTop:"1px solid #2d0d3d",padding:"10px 16px"}}>
        <div style={{fontSize:10,fontWeight:700,color:C.purple,letterSpacing:1,marginBottom:8}}>
          CREDIT SPREAD — BCS / BPS &nbsp;<span style={{fontWeight:400,color:"#444"}}>(optional — auto-adds to day total)</span>
        </div>
        <div style={{display:"flex",gap:8,flexWrap:"wrap",alignItems:"flex-end"}}>
          {[
            {lb:"TYPE",el:(
              <select value={day.spread.type} disabled={locked} onChange={e=>updSpread("type",e.target.value)}
                style={{...SINP,border:"1px solid #4a0e6a"}}>
                <option value="">— None —</option><option>BCS</option><option>BPS</option>
              </select>
            )},
            {lb:"TIME IN",         el:<input type="text"   placeholder="9:45 AM"  value={day.spread.timeIn}      disabled={locked} onChange={e=>updSpread("timeIn",e.target.value)}      style={SINP}/>},
            {lb:"TIME OUT",        el:<input type="text"   placeholder="10:30 AM" value={day.spread.timeOut}     disabled={locked} onChange={e=>updSpread("timeOut",e.target.value)}     style={SINP}/>},
            {lb:"STRIKES",         el:<input type="text"   placeholder="445/450"  value={day.spread.strikes}     disabled={locked} onChange={e=>updSpread("strikes",e.target.value)}     style={{...SINP,width:76}}/>},
            {lb:"CREDIT RECV'D $", el:<input type="number" placeholder="0.00"     value={day.spread.creditRecvd} disabled={locked} onChange={e=>updSpread("creditRecvd",e.target.value)} style={SINP}/>},
            {lb:"MAX RISK $",      el:<input type="number" placeholder="0.00"     value={day.spread.maxRisk}     disabled={locked} onChange={e=>updSpread("maxRisk",e.target.value)}     style={SINP}/>},
            {lb:"CONTRACTS",       el:<input type="number" placeholder="0"        value={day.spread.contracts}   disabled={locked} onChange={e=>updSpread("contracts",e.target.value)}   style={{...SINP,width:70}}/>},
            {lb:"OUTCOME $",       el:<input type="number" placeholder="0.00"     value={day.spread.outcome}     disabled={locked} onChange={e=>updSpread("outcome",e.target.value)}
              style={{...SINP,borderColor:spreadOut>0?C.green:spreadOut<0?C.red:"#4a0e6a",color:spreadOut>0?C.green:spreadOut<0?C.red:"#e0e0e0"}}/>},
            {lb:"NOTES / REF",     el:<input type="text"   placeholder="screenshot ref" value={day.spread.notes} disabled={locked} onChange={e=>updSpread("notes",e.target.value)}      style={{...SINP,width:140}}/>},
          ].map(({lb,el})=>(
            <div key={lb} style={{display:"flex",flexDirection:"column",gap:3}}>
              <span style={{fontSize:9,color:C.purple,fontWeight:700,letterSpacing:1,whiteSpace:"nowrap"}}>{lb}</span>
              {el}
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

// ─── Rules & Guide ────────────────────────────────────────────────────────────
function RulesGuide(){
  const [open,setOpen]=useState(null);
  const sections=[
    {icon:"🎯",title:"Overall Goal",color:"#D4A017",lines:[
      "Achieve $8,000 total profit over 3 weeks by scaling position size progressively.",
      "Week 1 → 1 contract | Daily $100 | Weekly $500",
      "Week 2 → 3 contracts | Daily $500 | Weekly $2,500 | Cumulative $3,000",
      "Week 3 → 5 contracts | Daily $1,000 | Weekly $5,000 | Final Total $8,000",
    ]},
    {icon:"⚙️",title:"Required Trade Data — Log Every Trade",color:"#0D47A1",lines:[
      "Log for every trade: Date, Entry Time, Call/Put, Contracts, Entry Price $, Exit Price $",
      "Gross P&L auto-calculates: (Exit − Entry) × Contracts × 100",
      "Rules Met? → Yes / Partial / No — be honest with yourself every single time",
      "Screenshot of order book → enter the file name in the Screenshot column",
    ]},
    {icon:"📊",title:"Scaling Out of Positions",color:"#6A1E6A",lines:[
      "Bought 5 contracts and want to exit 1 at a time? Use one trade row per exit.",
      "Each row: same Entry Time & Entry Price, adjust Contracts to the partial size, enter the Exit Price for that tranche.",
      "Example: Entry $2.00 × 5 contracts bought",
      "  → Row 1: 2 contracts exited @ $2.50  =  +$100",
      "  → Row 2: 2 contracts exited @ $2.80  =  +$160",
      "  → Row 3: 1 contract exited @ $3.10   =  +$110",
      "Hit '＋ Add Trade Row' at the bottom of any day to add as many rows as needed.",
    ]},
    {icon:"📸",title:"Proof of Execution — Non-Negotiable",color:"#1B5E20",lines:[
      "Screenshot your order book showing entry + exit for EVERY trade.",
      "FLAT DAY RULE: Made no trades? You STILL must screenshot your order book and log it.",
      "If it is not documented with a screenshot reference, it does not count toward the challenge.",
    ]},
    {icon:"🛑",title:"3-Loss Rule — Protect Your Drawdown",color:"#FF1744",lines:[
      "If you take 3 losing trades in a single day, you are IMMEDIATELY done for the day.",
      "The tracker flags this automatically when 3 rows show negative P&L.",
      "DO NOT take a 4th trade trying to recover. You will only make it worse.",
      "Close your platform. Review what happened. Come back tomorrow fresh.",
    ]},
    {icon:"💳",title:"Credit Spreads — BCS / BPS Rules",color:"#9C27B0",lines:[
      "Deploy a Bull Call Spread (BCS) or Bull Put Spread (BPS) only when a clear, high-probability setup appears.",
      "Log: Type, Time In, Time Out, Strikes, Credit Received $, Max Risk $, Contracts, Outcome $.",
      "Spread Outcome $ is automatically added into your Daily Total P&L.",
      "MAXIMUM risk capital: $5,000 across the position.",
      "MAXIMUM spread width: $5 wide. MAXIMUM contracts: 10.",
      "Do NOT force spreads. Only deploy when ALL rules and confirmations are met.",
    ]},
    {icon:"⚖️",title:"Core Rules — Non-Negotiable Every Day",color:"#FFB300",lines:[
      "1.  ONE TRADE MENTALITY — One clean setup per day is enough. Quality over quantity.",
      "2.  NO FORCING TRADES — Nothing lines up? Stay flat. No setup = no trade.",
      "3.  FOLLOW ALL CONFIRMATIONS — Trend + MAs + DMI + TTM must ALL align before entry.",
      "4.  TAKE PROFIT WHEN AVAILABLE — Don't get greedy. Base hits beat home runs.",
      "5.  CUT LOSSES WHEN RULES BREAK — Momentum fades or setup is invalid? Exit. No hoping.",
      "6.  ADJUST FOR CONDITIONS — High volatility: size down. Chop: don't trade at all.",
      "7.  STAY EMOTIONALLY NEUTRAL — No revenge trading. No overtrading after wins or losses.",
    ]},
    {icon:"🚫",title:"Automatic Disqualifiers",color:"#FF1744",lines:[
      "❌  Using more contracts than your assigned week allows",
      "❌  Exceeding $5,000 max risk on any credit spread",
      "❌  Missing documentation or screenshot for any trade",
      "❌  Taking a 4th trade after 3 losses in one day",
      "❌  Entering trades that do not meet all confirmations",
    ]},
    {icon:"🏁",title:"How to Pass the Challenge",color:"#00C853",lines:[
      "✅  Hit $8,000 cumulative profit by end of Week 3",
      "✅  Follow all rules consistently across all 3 weeks",
      "✅  Screenshot proof for every trade — no exceptions",
      "✅  Show discipline in execution — not just in results",
      "",
      "The goal is not just to pass the challenge.",
      "The goal is to become the trader who can REPEAT it. 🔥",
    ]},
  ];
  return (
    <div>
      <div style={{marginBottom:20}}>
        <div style={{fontSize:20,fontWeight:900,color:C.gold,letterSpacing:-.5,marginBottom:4}}>📋 Rules & Challenge Guide</div>
        <div style={{fontSize:12,color:"#555"}}>Click any section to expand. Read this before you trade every day.</div>
      </div>
      <div style={{background:"#12121f",borderRadius:12,padding:16,marginBottom:20,border:"1px solid #1e1e2e"}}>
        <div style={{fontSize:10,fontWeight:700,color:"#444",letterSpacing:1,marginBottom:12}}>⚡ QUICK REFERENCE</div>
        <div style={{display:"grid",gridTemplateColumns:"repeat(auto-fill,minmax(220px,1fr))",gap:8}}>
          {[
            {t:"🧱 One clean trade per day is enough",c:"#666"},
            {t:"⛔ No setup = no trade — stay flat",c:"#666"},
            {t:"📸 Flat day still needs a screenshot",c:C.gold},
            {t:"🛑 3 losses = done trading for the day",c:C.red},
            {t:"📊 Scale out? Add a row per partial exit",c:C.purple},
            {t:"💰 Take profit when the market gives it",c:C.green},
            {t:"✂️ Cut losses the moment rules break",c:C.amber},
            {t:"💳 Max $5K risk on any credit spread",c:C.purple},
          ].map(({t,c})=>(
            <div key={t} style={{background:"#0d0d1a",borderRadius:8,padding:"8px 12px",fontSize:12,color:c,fontWeight:600,borderLeft:`3px solid ${c}44`}}>{t}</div>
          ))}
        </div>
      </div>
      <div style={{display:"flex",flexDirection:"column",gap:8}}>
        {sections.map((s,i)=>(
          <div key={i} style={{borderRadius:10,overflow:"hidden",border:`1px solid ${open===i?s.color+"55":"#1e1e2e"}`}}>
            <button onClick={()=>setOpen(open===i?null:i)}
              style={{width:"100%",background:open===i?`linear-gradient(135deg,${s.color}14,#12121f)`:"#12121f",border:"none",padding:"13px 16px",display:"flex",alignItems:"center",justifyContent:"space-between",cursor:"pointer",gap:10}}>
              <div style={{display:"flex",alignItems:"center",gap:10}}>
                <span style={{fontSize:18}}>{s.icon}</span>
                <span style={{fontSize:13,fontWeight:800,color:open===i?s.color:"#ccc",textAlign:"left"}}>{s.title}</span>
              </div>
              <span style={{color:open===i?s.color:"#333",fontSize:18,transition:"transform .2s",transform:open===i?"rotate(180deg)":"none",flexShrink:0}}>▾</span>
            </button>
            {open===i&&(
              <div style={{background:"#0a0a12",borderTop:`1px solid ${s.color}33`,padding:"14px 20px"}}>
                {s.lines.map((line,li)=>(
                  <div key={li} style={{padding:"5px 0",fontSize:13,color:line?"#bbb":"transparent",borderBottom:li<s.lines.length-1?"1px solid #131320":"none",paddingLeft:line.startsWith(" ")?20:0}}>
                    {line||"\u00a0"}
                  </div>
                ))}
              </div>
            )}
          </div>
        ))}
      </div>
    </div>
  );
}

// ─── Student Tracker ──────────────────────────────────────────────────────────
function StudentTracker({ name, data, onSave, isCoach }){
  const [local,setLocal]=useState(data);
  const [activeWeek,setActiveWeek]=useState(0);
  const [tab,setTab]=useState("tracker");
  const [saving,setSaving]=useState(false);
  const [saved,setSaved]=useState(false);

  useEffect(()=>setLocal(data),[data]);

  const total=calcChallengeTotal(local);
  const pct=Math.min(100,(total/CHALLENGE_GOAL)*100);
  const handleSave=async()=>{ setSaving(true); await onSave(name,local); setSaving(false); setSaved(true); setTimeout(()=>setSaved(false),2200); };
  const updateDay=(wi,di,nd)=>setLocal(prev=>({...prev,weeks:prev.weeks.map((w,wii)=>wii===wi?{...w,days:w.days.map((d,dii)=>dii===di?nd:d)}:w)}));
  const wk=WEEK_CONFIG[activeWeek];
  const wkData=local.weeks[activeWeek];
  const wkTotal=calcWeekTotal(wkData);

  return (
    <div>
      <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",marginBottom:20,flexWrap:"wrap",gap:12}}>
        <div>
          <div style={{fontSize:22,fontWeight:900,color:"#fff",letterSpacing:-.5}}>{name}</div>
          <div style={{fontSize:12,color:"#555",marginTop:2}}>Daily Profit Challenge</div>
        </div>
        <div style={{display:"flex",gap:8,alignItems:"center",flexWrap:"wrap"}}>
          {["tracker","rules"].map(t=>(
            <button key={t} onClick={()=>setTab(t)}
              style={{background:tab===t?C.gold:"#1a1a2e",color:tab===t?"#000":"#666",border:`1px solid ${tab===t?C.gold:"#2a2a3e"}`,borderRadius:8,padding:"8px 16px",fontWeight:700,fontSize:12,cursor:"pointer",transition:"all .15s"}}>
              {t==="tracker"?"📊 Tracker":"📋 Rules & Guide"}
            </button>
          ))}
          <button onClick={handleSave} disabled={saving}
            style={{background:saved?C.green:C.gold,color:"#000",border:"none",borderRadius:8,padding:"10px 22px",fontWeight:800,fontSize:13,cursor:"pointer",opacity:saving?.7:1,transition:"all .2s"}}>
            {saving?"Saving…":saved?"✅ Saved!":"💾 Save"}
          </button>
        </div>
      </div>

      {tab==="rules"&&<RulesGuide/>}
      {tab==="tracker"&&<>
        <div style={{background:"#12121f",borderRadius:12,padding:16,marginBottom:20,border:"1px solid #1e1e2e"}}>
          <div style={{display:"flex",gap:12,flexWrap:"wrap",marginBottom:10}}>
            <StatBox label="Challenge Total" value={fmt(total)} color={total>=CHALLENGE_GOAL?C.green:C.gold}/>
            <StatBox label="Target"          value={fmt(CHALLENGE_GOAL)} color="#444"/>
            <StatBox label="Remaining"       value={fmt(Math.max(0,CHALLENGE_GOAL-total))} color="#666"/>
            <StatBox label="Progress"        value={`${pct.toFixed(0)}%`} color={pct>=100?C.green:C.gold} sub={pct>=100?"🏆 CHALLENGE PASSED":""}/>
          </div>
          <ProgressBar value={total} max={CHALLENGE_GOAL} color={pct>=100?C.green:C.gold}/>
        </div>
        <div style={{display:"flex",gap:8,marginBottom:16}}>
          {WEEK_CONFIG.map((w,i)=>{
            const wt=calcWeekTotal(local.weeks[i]);
            const passed=wt>=w.weeklyTarget;
            const clr=[C.week1,C.week2,C.week3][i];
            return (
              <button key={i} onClick={()=>setActiveWeek(i)}
                style={{flex:1,padding:"10px 8px",borderRadius:10,border:"none",cursor:"pointer",background:activeWeek===i?clr:"#12121f",color:activeWeek===i?"#fff":"#555",fontWeight:700,fontSize:12,transition:"all .2s",borderBottom:activeWeek===i?`3px solid ${C.gold}`:"3px solid transparent"}}>
                <div>Week {w.week}</div>
                <div style={{fontSize:10,opacity:.8,fontFamily:"'DM Mono',monospace"}}>{fmt(wt)}</div>
                {passed&&<div style={{fontSize:9,color:C.green}}>✅ PASSED</div>}
              </button>
            );
          })}
        </div>
        <div style={{display:"flex",gap:10,flexWrap:"wrap",marginBottom:10}}>
          <StatBox label="Week P&L"   value={fmt(wkTotal)}                            color={wkTotal>=wk.weeklyTarget?C.green:C.gold}/>
          <StatBox label="Wk Target"  value={fmt(wk.weeklyTarget)}                    color="#444"/>
          <StatBox label="Remaining"  value={fmt(Math.max(0,wk.weeklyTarget-wkTotal))} color="#666"/>
          <StatBox label="Daily Goal" value={fmt(wk.dailyGoal)}                       color="#555"/>
          <StatBox label="Contracts"  value={wk.contracts}                            color="#555" sub={wk.label}/>
        </div>
        <div style={{marginBottom:20}}>
          <ProgressBar value={wkTotal} max={wk.weeklyTarget} color={[C.week1,C.week2,C.week3][activeWeek]}/>
        </div>
        {wkData.days.map((day,di)=>(
          <DayBlock key={di} dayIdx={di} day={day}
            onChange={nd=>updateDay(activeWeek,di,nd)}
            weekConfig={wk} locked={false}/>
        ))}
      </>}
    </div>
  );
}

// ─── Roster Manager (inside Coach Dashboard) ──────────────────────────────────
function RosterManager({ students, onAdd, onRemove }){
  const [newName,setNewName]=useState("");
  const [err,setErr]=useState("");
  const [confirmRemove,setConfirmRemove]=useState(null);

  const handleAdd=()=>{
    const name=newName.trim();
    if(!name){ setErr("Enter a name"); return; }
    if(students.map(s=>s.toLowerCase()).includes(name.toLowerCase())){ setErr("Student already exists"); return; }
    onAdd(name); setNewName(""); setErr("");
  };

  const handleRemoveClick=(name)=>{
    if(confirmRemove===name){ onRemove(name); setConfirmRemove(null); }
    else { setConfirmRemove(name); }
  };

  return (
    <div style={{background:"#12121f",borderRadius:12,border:"1px solid #D4A01733",marginBottom:24,overflow:"hidden"}}>
      {/* Header */}
      <div style={{padding:"14px 20px",borderBottom:"1px solid #1e1e2e",display:"flex",alignItems:"center",justifyContent:"space-between",flexWrap:"wrap",gap:8}}>
        <div style={{display:"flex",alignItems:"center",gap:10}}>
          <span style={{fontWeight:800,fontSize:13,color:C.gold}}>👥 Manage Roster</span>
          <span style={{fontSize:11,color:"#444"}}>Add or remove students from the challenge</span>
        </div>
        <span style={{fontSize:11,color:"#555",fontFamily:"'DM Mono',monospace"}}>{students.length} student{students.length!==1?"s":""} enrolled</span>
      </div>

      <div style={{padding:"16px 20px"}}>
        {/* Current roster pills */}
        <div style={{display:"flex",flexWrap:"wrap",gap:8,marginBottom:16,minHeight:36}}>
          {students.length===0&&(
            <span style={{fontSize:12,color:"#444",fontStyle:"italic",alignSelf:"center"}}>No students enrolled yet. Add one below.</span>
          )}
          {students.map(name=>{
            const isConfirming=confirmRemove===name;
            return (
              <div key={name} style={{display:"flex",alignItems:"stretch",background:isConfirming?"#200808":"#1a1a2e",border:`1px solid ${isConfirming?C.red+"88":"#2a2a3e"}`,borderRadius:22,overflow:"hidden",transition:"all .2s"}}>
                <span style={{padding:"6px 14px",fontSize:13,fontWeight:700,color:isConfirming?C.red:"#ddd",lineHeight:"20px"}}>{name}</span>
                <button
                  onClick={()=>handleRemoveClick(name)}
                  title={isConfirming?"Click again to confirm removal":"Remove student"}
                  style={{background:isConfirming?C.red+"22":"transparent",border:"none",borderLeft:`1px solid ${isConfirming?C.red+"66":"#2a2a3e"}`,padding:"6px 11px",cursor:"pointer",color:isConfirming?C.red:"#555",fontSize:isConfirming?10:16,fontWeight:isConfirming?800:400,transition:"all .2s",lineHeight:"20px"}}
                  onMouseEnter={e=>{ if(!isConfirming) e.currentTarget.style.color=C.red; }}
                  onMouseLeave={e=>{ if(!isConfirming) e.currentTarget.style.color="#555"; }}>
                  {isConfirming?"CONFIRM?":"×"}
                </button>
              </div>
            );
          })}
        </div>

        {/* Confirm warning */}
        {confirmRemove&&(
          <div style={{background:"#1a1000",border:"1px solid #D4A01744",borderRadius:8,padding:"8px 14px",marginBottom:14,display:"flex",alignItems:"center",justifyContent:"space-between",flexWrap:"wrap",gap:8}}>
            <span style={{fontSize:12,color:C.amber}}>
              ⚠️ Click <strong>CONFIRM?</strong> again to remove <strong>{confirmRemove}</strong>. Their trade data stays saved and can be restored if re-added.
            </span>
            <button onClick={()=>setConfirmRemove(null)}
              style={{background:"none",border:"1px solid #444",borderRadius:6,padding:"4px 10px",color:"#666",fontSize:11,cursor:"pointer"}}>
              Cancel
            </button>
          </div>
        )}

        {/* Add student */}
        <div style={{display:"flex",gap:8,alignItems:"center",flexWrap:"wrap"}}>
          <input
            type="text"
            placeholder="New student name..."
            value={newName}
            onChange={e=>{ setNewName(e.target.value); setErr(""); }}
            onKeyDown={e=>e.key==="Enter"&&handleAdd()}
            style={{background:"#0d0d1a",border:`1px solid ${err?C.red:C.gold+"66"}`,borderRadius:8,padding:"9px 14px",color:"#e0e0e0",fontSize:13,fontFamily:"'DM Mono',monospace",outline:"none",width:220,transition:"border-color .2s"}}
          />
          <button onClick={handleAdd}
            style={{background:C.gold,color:"#000",border:"none",borderRadius:8,padding:"9px 20px",fontWeight:800,fontSize:13,cursor:"pointer",transition:"opacity .15s"}}
            onMouseEnter={e=>e.currentTarget.style.opacity=".85"}
            onMouseLeave={e=>e.currentTarget.style.opacity="1"}>
            ＋ Add Student
          </button>
          {err&&<span style={{fontSize:12,color:C.red,fontWeight:600}}>{err}</span>}
        </div>
      </div>
    </div>
  );
}

// ─── Coach Dashboard ──────────────────────────────────────────────────────────
function CoachDashboard({ allData, students, onAddStudent, onRemoveStudent }){
  const [sortBy,setSortBy]=useState("total");

  const rows=students.map(name=>{
    const d=allData[name]||emptyStudent();
    const total=calcChallengeTotal(d);
    const weeks=WEEK_CONFIG.map((wc,i)=>{ const wt=calcWeekTotal(d.weeks[i]); return {total:wt,passed:wt>=wc.weeklyTarget}; });
    return { name, total, weeks, passed:total>=CHALLENGE_GOAL };
  });
  const sorted=[...rows].sort((a,b)=>sortBy==="total"?b.total-a.total:a.name.localeCompare(b.name));
  const leaderTotal=rows.length?Math.max(...rows.map(r=>r.total)):0;
  const avgTotal=rows.length?rows.reduce((s,r)=>s+r.total,0)/rows.length:0;
  const passedCount=rows.filter(r=>r.passed).length;

  return (
    <div>
      <div style={{marginBottom:24}}>
        <div style={{fontSize:22,fontWeight:900,color:C.gold,letterSpacing:-.5,marginBottom:4}}>🎯 Coach Dashboard</div>
        <div style={{fontSize:12,color:"#555"}}>Real-time overview — all students</div>
      </div>

      {/* KPIs */}
      <div style={{display:"flex",gap:12,flexWrap:"wrap",marginBottom:24}}>
        <StatBox label="Students"      value={students.length} color={C.gold}/>
        <StatBox label="Passed"        value={passedCount}     color={C.green} sub={`${students.length?((passedCount/students.length)*100).toFixed(0):0}% pass rate`}/>
        <StatBox label="Class Average" value={fmt(avgTotal)}   color="#888"/>
        <StatBox label="Top Performer" value={fmt(leaderTotal)} color={C.gold} sub={rows.find(r=>r.total===leaderTotal)?.name||"—"}/>
      </div>

      {/* Roster Manager */}
      <RosterManager students={students} onAdd={onAddStudent} onRemove={onRemoveStudent}/>

      {/* Leaderboard */}
      <div style={{background:"#12121f",borderRadius:14,border:"1px solid #1e1e2e",overflow:"hidden",marginBottom:24}}>
        <div style={{padding:"14px 20px",borderBottom:"1px solid #1e1e2e",display:"flex",justifyContent:"space-between",alignItems:"center"}}>
          <span style={{fontWeight:800,fontSize:13,color:"#ccc"}}>📊 Leaderboard</span>
          <div style={{display:"flex",gap:6}}>
            {[["total","By P&L"],["name","By Name"]].map(([s,l])=>(
              <button key={s} onClick={()=>setSortBy(s)}
                style={{background:sortBy===s?C.gold:"#1a1a2e",color:sortBy===s?"#000":"#666",border:"none",borderRadius:6,padding:"4px 12px",fontSize:11,fontWeight:700,cursor:"pointer"}}>
                {l}
              </button>
            ))}
          </div>
        </div>
        {rows.length===0?(
          <div style={{padding:"32px",textAlign:"center",color:"#444",fontSize:13,fontStyle:"italic"}}>
            No students enrolled yet. Add students using the roster panel above.
          </div>
        ):(
          <div style={{overflowX:"auto"}}>
            <table style={{width:"100%",borderCollapse:"collapse",minWidth:580}}>
              <thead><tr style={{background:"#0d0d1a"}}>{["RANK","STUDENT","WK 1","WK 2","WK 3","TOTAL P&L","PROGRESS","STATUS"].map(h=><th key={h} style={TH}>{h}</th>)}</tr></thead>
              <tbody>
                {sorted.map((row,i)=>{
                  const p=Math.min(100,(row.total/CHALLENGE_GOAL)*100);
                  return (
                    <tr key={row.name} style={{background:i%2===0?"#0f0f1c":"#12121f",borderLeft:`3px solid ${row.passed?C.green:"#1e1e2e"}`}}>
                      <td style={{...TD,color:i===0?C.gold:"#555",fontWeight:800,fontSize:16}}>{i===0?"🥇":i===1?"🥈":i===2?"🥉":i+1}</td>
                      <td style={{...TD,fontWeight:800,color:"#ddd"}}>{row.name}</td>
                      {row.weeks.map((w,wi)=>(
                        <td key={wi} style={{...TD,fontFamily:"'DM Mono',monospace",fontSize:12,color:w.passed?C.green:w.total<0?C.red:"#666"}}>{fmt(w.total)}{w.passed?" ✅":""}</td>
                      ))}
                      <td style={{...TD,fontFamily:"'DM Mono',monospace",fontWeight:800,fontSize:14,color:row.passed?C.green:row.total>0?C.gold:"#555"}}>{fmt(row.total)}</td>
                      <td style={{...TD,minWidth:110}}>
                        <ProgressBar value={row.total} max={CHALLENGE_GOAL} color={row.passed?C.green:C.gold}/>
                        <span style={{fontSize:10,color:"#444",fontFamily:"'DM Mono',monospace"}}>{p.toFixed(0)}%</span>
                      </td>
                      <td style={TD}><Badge text={row.passed?"🏆 PASSED":row.total>0?"⏳ IN PROGRESS":"❌ NO TRADES"} bg={row.passed?C.green+"33":row.total>0?C.gold+"22":"#ff000022"} color={row.passed?C.green:row.total>0?C.gold:C.red}/></td>
                    </tr>
                  );
                })}
              </tbody>
            </table>
          </div>
        )}
      </div>

      {/* Weekly cards */}
      {rows.length>0&&<>
        <div style={{fontSize:13,fontWeight:800,color:"#ccc",marginBottom:12}}>Weekly Breakdown — All Students</div>
        <div style={{display:"grid",gridTemplateColumns:"repeat(auto-fill,minmax(250px,1fr))",gap:12}}>
          {rows.map(row=>{
            const p=Math.min(100,(row.total/CHALLENGE_GOAL)*100);
            return (
              <div key={row.name} style={{background:"#12121f",borderRadius:12,padding:16,border:`1px solid ${row.passed?C.green+"44":"#1e1e2e"}`}}>
                <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:10}}>
                  <span style={{fontWeight:800,color:"#ddd",fontSize:14}}>{row.name}</span>
                  <Badge text={row.passed?"🏆 PASSED":`${p.toFixed(0)}%`} bg={row.passed?C.green+"33":C.gold+"22"} color={row.passed?C.green:C.gold}/>
                </div>
                <ProgressBar value={row.total} max={CHALLENGE_GOAL} color={row.passed?C.green:C.gold}/>
                <div style={{marginTop:10,display:"grid",gridTemplateColumns:"1fr 1fr 1fr",gap:6}}>
                  {row.weeks.map((w,wi)=>(
                    <div key={wi} style={{background:"#0d0d1a",borderRadius:8,padding:"8px 6px",textAlign:"center"}}>
                      <div style={{fontSize:9,color:"#444",marginBottom:2}}>WK {wi+1}</div>
                      <div style={{fontFamily:"'DM Mono',monospace",fontSize:11,fontWeight:700,color:w.passed?C.green:w.total<0?C.red:"#666"}}>{fmt(w.total)}</div>
                      {w.passed&&<div style={{fontSize:9,color:C.green}}>✅</div>}
                    </div>
                  ))}
                </div>
                <div style={{marginTop:8,textAlign:"right",fontFamily:"'DM Mono',monospace",fontWeight:800,fontSize:14,color:row.passed?C.green:C.gold}}>{fmt(row.total)}</div>
              </div>
            );
          })}
        </div>
      </>}
    </div>
  );
}

// ─── Login ────────────────────────────────────────────────────────────────────
function LoginScreen({ onLogin, students }){
  const [sel,setSel]=useState("");
  const [pin,setPin]=useState("");
  const [err,setErr]=useState("");
  const go=()=>{
    if(sel==="COACH"){ if(pin===COACH_PIN){onLogin("COACH");setErr("");}else setErr("Incorrect PIN"); }
    else if(sel) onLogin(sel);
    else setErr("Please select your name");
  };
  return (
    <div style={{minHeight:"100vh",background:C.dark,display:"flex",alignItems:"center",justifyContent:"center",fontFamily:"'DM Sans',sans-serif",backgroundImage:"radial-gradient(ellipse at 20% 50%,#0f3460 0%,transparent 60%)"}}>
      <div style={{width:460,maxWidth:"92vw"}}>
        <div style={{textAlign:"center",marginBottom:36}}>
          <div style={{fontSize:48,marginBottom:8}}>📈</div>
          <div style={{fontSize:30,fontWeight:900,color:C.gold,letterSpacing:-1}}>Take Profit Trading</div>
          <div style={{fontSize:13,color:"#444",marginTop:4}}>Daily Profit Challenge Tracker</div>
        </div>
        <div style={{background:"#12121f",borderRadius:16,padding:32,border:"1px solid #1e1e2e"}}>
          <div style={{fontSize:11,fontWeight:700,color:"#444",marginBottom:14,letterSpacing:1}}>SELECT YOUR NAME</div>
          {students.length===0?(
            <div style={{fontSize:12,color:"#444",fontStyle:"italic",padding:"12px 0",marginBottom:12}}>No students enrolled yet. Ask your coach to add you.</div>
          ):(
            <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:8,marginBottom:16}}>
              {students.map(s=>(
                <button key={s} onClick={()=>{setSel(s);setPin("");setErr("");}}
                  style={{background:sel===s?C.gold:"#1a1a2e",color:sel===s?"#000":"#888",border:`1px solid ${sel===s?C.gold:"#2a2a3e"}`,borderRadius:10,padding:"12px 8px",fontWeight:700,fontSize:14,cursor:"pointer",transition:"all .15s"}}>
                  {s}
                </button>
              ))}
            </div>
          )}
          <div style={{borderTop:"1px solid #1e1e2e",paddingTop:14,marginBottom:12}}>
            <button onClick={()=>{setSel("COACH");setPin("");setErr("");}}
              style={{width:"100%",background:sel==="COACH"?"#0f3460":"#1a1a2e",color:sel==="COACH"?C.gold:"#555",border:`1px solid ${sel==="COACH"?C.gold+"88":"#2a2a3e"}`,borderRadius:10,padding:"12px 8px",fontWeight:700,fontSize:13,cursor:"pointer"}}>
              🎓 Coach Login
            </button>
          </div>
          {sel==="COACH"&&(
            <input type="password" placeholder="Enter coach PIN" value={pin}
              onChange={e=>{setPin(e.target.value);setErr("");}} onKeyDown={e=>e.key==="Enter"&&go()}
              style={{width:"100%",background:"#0d0d1a",border:"1px solid #D4A01744",borderRadius:10,padding:"12px 16px",color:"#e0e0e0",fontSize:14,fontFamily:"'DM Mono',monospace",outline:"none",marginBottom:12}}/>
          )}
          {err&&<div style={{color:C.red,fontSize:12,marginBottom:10,textAlign:"center"}}>{err}</div>}
          <button onClick={go}
            style={{width:"100%",background:`linear-gradient(135deg,${C.gold},#f0c040)`,color:"#000",border:"none",borderRadius:10,padding:14,fontWeight:900,fontSize:15,cursor:"pointer"}}>
            {sel==="COACH"?"🎓 Enter Coach Dashboard":sel?`Enter as ${sel}`:"Select Your Name Above"}
          </button>

        </div>
      </div>
    </div>
  );
}

// ─── App ──────────────────────────────────────────────────────────────────────
export default function App(){
  const [user,setUser]=useState(null);
  const [allData,setAllData]=useState({});
  const [students,setStudents]=useState([]);
  const [loading,setLoading]=useState(false);
  const [activeStudent,setActiveStudent]=useState(null);
  const [view,setView]=useState("dashboard");

  // Pre-load roster so login screen shows current student list
  useEffect(()=>{ loadRoster().then(r=>setStudents(r)); },[]);

  const login=async who=>{
    setLoading(true);
    const roster=await loadRoster();
    setStudents(roster);
    const d=await loadAll(roster);
    setAllData(d); setUser(who); setLoading(false);
  };

  const handleSave=async(name,data)=>{
    await saveStudent(name,data);
    setAllData(prev=>({...prev,[name]:data}));
  };

  const handleAddStudent=async name=>{
    const updated=[...students,name];
    setStudents(updated);
    await saveRoster(updated);
    setAllData(prev=>({...prev,[name]:emptyStudent()}));
  };

  const handleRemoveStudent=async name=>{
    const updated=students.filter(s=>s!==name);
    setStudents(updated);
    await saveRoster(updated);
    if(activeStudent===name){ setActiveStudent(null); setView("dashboard"); }
    setAllData(prev=>{ const n={...prev}; delete n[name]; return n; });
  };

  if(!user) return <LoginScreen onLogin={login} students={students}/>;

  if(loading) return (
    <div style={{minHeight:"100vh",background:C.dark,display:"flex",alignItems:"center",justifyContent:"center"}}>
      <span style={{color:C.gold,fontSize:18,fontFamily:"'DM Mono',monospace"}}>Loading...</span>
    </div>
  );

  const isCoach=user==="COACH";

  return (
    <div style={{minHeight:"100vh",background:C.dark,color:"#e0e0e0",fontFamily:"'DM Sans',sans-serif"}}>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;700;800;900&family=DM+Mono:wght@400;500;700&display=swap');
        *{box-sizing:border-box} input,select{box-sizing:border-box}
        input:focus,select:focus{border-color:#D4A017!important;box-shadow:0 0 0 2px #D4A01722}
        ::-webkit-scrollbar{width:6px;height:6px}
        ::-webkit-scrollbar-track{background:#0d0d1a}
        ::-webkit-scrollbar-thumb{background:#2a2a3e;border-radius:3px}
      `}</style>

      {/* Nav */}
      <div style={{background:"#0d0d1a",borderBottom:"1px solid #1e1e2e",padding:"0 20px",display:"flex",alignItems:"center",justifyContent:"space-between",height:52,position:"sticky",top:0,zIndex:100}}>
        <div style={{display:"flex",alignItems:"center",gap:8}}>
          <span style={{fontSize:18}}>📈</span>
          <span style={{fontWeight:900,color:C.gold,fontSize:14}}>Take Profit Trading</span>
          <span style={{fontSize:10,color:"#2a2a2a",marginLeft:2}}>Daily Profit Challenge</span>
        </div>
        <div style={{display:"flex",alignItems:"center",gap:5,flexWrap:"wrap"}}>
          {isCoach?(
            <>
              <button onClick={()=>{setView("dashboard");setActiveStudent(null);}}
                style={{background:view==="dashboard"?C.gold:"transparent",color:view==="dashboard"?"#000":"#444",border:"none",borderRadius:6,padding:"5px 12px",fontWeight:700,fontSize:11,cursor:"pointer"}}>
                Dashboard
              </button>
              {students.map(s=>(
                <button key={s} onClick={()=>{setView("student");setActiveStudent(s);}}
                  style={{background:activeStudent===s&&view==="student"?"#0f3460":"transparent",color:activeStudent===s&&view==="student"?C.gold:"#444",border:"none",borderRadius:6,padding:"5px 9px",fontWeight:700,fontSize:11,cursor:"pointer"}}>
                  {s}
                </button>
              ))}
            </>
          ):(
            <span style={{fontSize:12,color:C.gold,fontWeight:700}}>👤 {user}</span>
          )}
          <button onClick={()=>setUser(null)} style={{background:"#1a1a2e",color:"#444",border:"1px solid #1e1e2e",borderRadius:6,padding:"5px 10px",fontSize:10,cursor:"pointer",marginLeft:4}}>
            Sign Out
          </button>
        </div>
      </div>

      {/* Content */}
      <div style={{maxWidth:1200,margin:"0 auto",padding:"24px 20px"}}>
        {isCoach
          ? view==="dashboard"
            ? <CoachDashboard
                allData={allData}
                students={students}
                onAddStudent={handleAddStudent}
                onRemoveStudent={handleRemoveStudent}/>
            : activeStudent&&<StudentTracker
                name={activeStudent}
                data={allData[activeStudent]||emptyStudent()}
                onSave={handleSave}
                isCoach/>
          : <StudentTracker
              name={user}
              data={allData[user]||emptyStudent()}
              onSave={handleSave}
              isCoach={false}/>
        }
      </div>
    </div>
  );
}
