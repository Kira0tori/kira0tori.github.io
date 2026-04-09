# kira0tori.github.io

<script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
<script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
<div id="root"></div>
<script>
const { useState, useEffect } = React;
const G = "#BA7517";
const GB = "rgba(186,117,23,0.12)";
const STATS = ["AGI","CON","FOR","PER","CHA","INT","VOL"];
const CLIST = "cof2_cl";
const ckey = n => "cof2_" + n.replace(/[^\w]/g,"_").slice(0,80);

function freshChar(nom="") {
  const d = { nom, joueur:"", niv:"" };
  STATS.forEach(s => { d[`${s}_v`]=""; d[`${s}_n`]=""; });
  d.vdp_name="";
  for(let i=1;i<=5;i++) { d[`vdp${i}_c`]=false; d[`vdp${i}_t`]=""; }
  d.init=""; d.def="";
  d.pv_max=""; d.pv=Array(12).fill(false);
  d.pc_max=""; d.pc=Array(12).fill(false);
  d.dr_type=""; d.dr_max=""; d.pm_max="";
  d.famille=""; d.profil=""; d.ideal=""; d.travers="";
  for(const k of ["c","d","m"]) { d[`att_${k}_t`]=""; d[`att_${k}_n`]=""; d[`att_${k}_m`]=""; }
  for(let i=0;i<3;i++) { d[`arme${i}_n`]=""; d[`arme${i}_a`]=""; d[`arme${i}_d`]=""; d[`arme${i}_s`]=""; }
  d.equip=""; d.desc="";
  for(let v=1;v<=6;v++) {
    d[`v${v}_n`] = v===6?"Prestige":"";
    for(let r=1;r<=5;r++) { d[`v${v}r${r}_c`]=false; d[`v${v}r${r}_t`]=""; }
  }
  return d;
}

const storage = {
  async get(key) {
    return { value: localStorage.getItem(key) };
  },
  async set(key, value) {
    localStorage.setItem(key, value);
  },
  async delete(key) {
    localStorage.removeItem(key);
  }
};

function App() {
  const [chars, setChars] = useState([]);
  const [cur, setCur] = useState(null);
  const [data, setData] = useState(null);
  const [tab, setTab] = useState(0);
  const [newName, setNewName] = useState("");
  const [creating, setCreating] = useState(false);
  const [loaded, setLoaded] = useState(false);
  const [saved, setSaved] = useState(true);

  useEffect(() => {
    (async () => {
      try { const r = await storage.get(CLIST); if(r && r.value) setChars(JSON.parse(r.value)); else {setChars([]);}} catch {console.error(e);}
      setLoaded(true);
    })();
  }, []);

useEffect(() => {
  if (!cur) return;

  (async () => {
    try {
      const r = await storage.get(ckey(cur));
      if (r && r.value) {
        setData(JSON.parse(r.value));
      } else {
        setData(freshChar(cur));
      }
    } catch (e) {
      console.error(e);
      setData(freshChar(cur));
    }
  })();
}, [cur]);

  useEffect(() => {
    if(!data||!cur) return;
    setSaved(false);
    const t = setTimeout(async()=>{
      try { await storage.set(ckey(cur), JSON.stringify(data)); setSaved(true); } catch {console.error(e);}
    }, 700);
    return ()=>clearTimeout(t);
  }, [data, cur]);

  const set = (k,v) => setData(p=>({...p,[k]:v}));
  const toggleDiamond = (arr,i) => set(arr, data[arr].map((x,j)=>j===i?!x:x));

  const createChar = async () => {
    const n = newName.trim();
    if(!n) return;
    const list = [...chars, n];
    await storage.set(CLIST, JSON.stringify(list));
    const cd = freshChar(n);
    await storage.set(ckey(n), JSON.stringify(cd));
    setChars(list); setCur(n); setData(cd); setNewName(""); setCreating(false);
  };

  const delChar = async (n) => {
    if(!confirm(`Supprimer "${n}" définitivement ?`)) return;
    const list = chars.filter(x=>x!==n);
    await storage.set(CLIST, JSON.stringify(list));
    try { await storage.delete(ckey(n)); } catch {console.error(e);}
    setChars(list);
    if(cur===n) { setCur(null); setData(null); }
  };

  if(!loaded) return React.createElement("div",{style:{padding:"2rem",color:"var(--color-text-secondary)"}}, "Chargement…");

  if(!cur) {
    return React.createElement("div",{style:{padding:"1.5rem",maxWidth:460,margin:"0 auto"}},
      React.createElement("div",{style:{fontFamily:"var(--font-serif)",fontSize:22,color:G,marginBottom:"0.5rem",borderBottom:`2px solid ${G}`,paddingBottom:10}}, "⚔ Chroniques Oubliées Fantasy"),
      React.createElement("p",{style:{fontSize:12,color:"var(--color-text-secondary)",marginBottom:16}}, "Fiches sauvegardées automatiquement dans ce navigateur."),
      chars.length===0 && !creating && React.createElement("p",{style:{fontSize:13,color:"var(--color-text-secondary)",marginBottom:12,fontStyle:"italic"}}, "Aucun personnage. Créez-en un pour commencer !"),
      ...chars.map(n=>React.createElement("div",{key:n,style:{display:"flex",alignItems:"center",gap:8,marginBottom:7,padding:"9px 13px",background:GB,borderRadius:8,border:`0.5px solid ${G}50`}},
        React.createElement("button",{onClick:()=>{setCur(n);setTab(0);},style:{flex:1,textAlign:"left",background:"none",border:"none",cursor:"pointer",fontSize:14,fontWeight:500,color:"var(--color-text-primary)",padding:0}}, n),
        React.createElement("span",{onClick:()=>delChar(n),style:{cursor:"pointer",fontSize:13,color:"var(--color-text-tertiary)",padding:"0 4px",lineHeight:1}}, "✕")
      )),
      creating
        ? React.createElement("div",{style:{display:"flex",gap:8,marginTop:10}},
            React.createElement("input",{autoFocus:true,value:newName,onChange:e=>setNewName(e.target.value),onKeyDown:e=>e.key==="Enter"&&createChar(),placeholder:"Nom du personnage…",style:{flex:1}}),
            React.createElement("button",{onClick:createChar,style:{background:G,color:"#fff",border:"none",borderRadius:6,padding:"6px 14px",cursor:"pointer",fontSize:13,fontWeight:500}}, "Créer"),
            React.createElement("button",{onClick:()=>setCreating(false)}, "Annuler")
          )
        : React.createElement("button",{onClick:()=>setCreating(true),style:{marginTop:10,background:G,color:"#fff",border:"none",borderRadius:6,padding:"8px 18px",cursor:"pointer",fontSize:13,fontWeight:500}}, "+ Nouveau personnage")
    );
  }

  if(!data) return React.createElement("div",{style:{padding:"2rem",color:"var(--color-text-secondary)"}}, "Chargement…");

  const lbl = (text,s={}) => React.createElement("div",{style:{fontSize:9,fontWeight:700,color:G,textTransform:"uppercase",letterSpacing:"0.08em",marginBottom:2,...s}},text);
  
  const box = (title, children, s={}) =>
    React.createElement("div",{style:{border:`0.5px solid ${G}45`,borderRadius:4,padding:"6px 8px",...s}},
      title && React.createElement("div",{style:{fontSize:10,fontWeight:700,color:G,textTransform:"uppercase",letterSpacing:"0.08em",borderBottom:`0.5px solid ${G}35`,marginBottom:5,paddingBottom:3}},title),
      children
    );

  const fi = (k,s={},ph="") =>
    React.createElement("input",{value:data[k]??"",onChange:e=>set(k,e.target.value),placeholder:ph,
      style:{background:"transparent",border:"none",borderBottom:`0.5px solid ${G}40`,color:"var(--color-text-primary)",padding:"1px 3px",fontSize:12,outline:"none",...s}});

  const diamonds = (arr,count=12) =>
    React.createElement("div",{style:{display:"flex",gap:2,flexWrap:"wrap",marginTop:3}},
      Array(count).fill(0).map((_,i)=>
        React.createElement("span",{key:i,onClick:()=>toggleDiamond(arr,i),style:{cursor:"pointer",fontSize:14,color:data[arr][i]?G:"var(--color-border-secondary)",userSelect:"none",lineHeight:1}},
          data[arr][i]?"◆":"◇")
      )
    );

  const renderVoieCol = (v) =>
    React.createElement("div",{key:v,style:{minWidth:0}},
      React.createElement("div",{style:{background:v<=3?G:"#6d5012",borderRadius:"3px 3px 0 0",padding:"3px 7px",marginBottom:4,display:"flex",alignItems:"center",gap:4}},
        React.createElement("span",{style:{fontSize:10,fontWeight:700,color:"#fff",letterSpacing:"0.05em",whiteSpace:"nowrap"}},v===6?"Prestige :":`Voie ${v} :`),
        React.createElement("input",{value:data[`v${v}_n`]??"",onChange:e=>set(`v${v}_n`,e.target.value),
          style:{background:"transparent",border:"none",fontSize:10,color:"#fff",flex:1,outline:"none",minWidth:0}})
      ),
      [1,2,3,4,5].map(r=>
        React.createElement("div",{key:r,style:{display:"flex",alignItems:"flex-start",gap:5,marginBottom:6,paddingBottom:4,borderBottom:`0.5px solid ${G}20`}},
          React.createElement("input",{type:"checkbox",checked:!!data[`v${v}r${r}_c`],onChange:e=>set(`v${v}r${r}_c`,e.target.checked),style:{marginTop:2,accentColor:G,flexShrink:0}}),
          React.createElement("div",{style:{flex:1,minWidth:0}},
            React.createElement("div",{style:{fontSize:10,color:G,fontWeight:600,marginBottom:1}},`Rang ${r} :`),
            React.createElement("input",{value:data[`v${v}r${r}_t`]??"",onChange:e=>set(`v${v}r${r}_t`,e.target.value),
              style:{width:"100%",background:"transparent",border:"none",borderBottom:`0.5px solid ${G}25`,fontSize:11,color:"var(--color-text-primary)",padding:"1px 0",boxSizing:"border-box",outline:"none"}})
          )
        )
      )
    );

  const renderPage1 = () =>
    React.createElement("div",null,
      // Header
      React.createElement("div",{style:{display:"flex",alignItems:"flex-end",gap:14,marginBottom:10,paddingBottom:8,borderBottom:`2px solid ${G}`}},
        React.createElement("div",{style:{fontFamily:"var(--font-serif)",fontSize:15,fontWeight:700,color:G,lineHeight:1.2,borderRight:`1px solid ${G}45`,paddingRight:14,minWidth:120}},
          "Chroniques","",React.createElement("br"),React.createElement("span",null,"Oubliées"),React.createElement("br"),React.createElement("span",{style:{fontSize:10,letterSpacing:"0.12em"}},"FANTASY")
        ),
        React.createElement("div",{style:{flex:1}},
          lbl("Nom du personnage",{marginBottom:2}),
          React.createElement("input",{value:data.nom??"",onChange:e=>set("nom",e.target.value),placeholder:"Nom du personnage…",
            style:{width:"100%",background:"transparent",border:"none",borderBottom:`2px solid ${G}`,fontSize:16,fontWeight:600,color:"var(--color-text-primary)",padding:"2px 4px",boxSizing:"border-box",outline:"none"}})
        )
      ),
      // Joueur/Niv
      React.createElement("div",{style:{display:"flex",gap:10,marginBottom:10}},
        React.createElement("div",{style:{flex:1}}, lbl("Joueur"), fi("joueur",{width:"100%"},"Nom du joueur")),
        React.createElement("div",null, lbl("Niveau"), fi("niv",{width:36,textAlign:"center"},"1"))
      ),
      // 3 columns
      React.createElement("div",{style:{display:"grid",gridTemplateColumns:"160px 1fr 185px",gap:10,marginBottom:10,minWidth:0}},
        // Col 1: Caractéristiques
        box("Caractéristiques",
          React.createElement("table",{style:{width:"100%",borderCollapse:"collapse"}},
            React.createElement("thead",null,
              React.createElement("tr",null,
                React.createElement("th",{style:{fontSize:9,color:G,textAlign:"left",paddingBottom:3,width:34,fontWeight:600}}),
                React.createElement("th",{style:{fontSize:9,color:G,textAlign:"center",paddingBottom:3,width:34,fontWeight:600}},"Val."),
                React.createElement("th",{style:{fontSize:9,color:G,textAlign:"left",paddingBottom:3,fontWeight:600}},"Notes")
              )
            ),
            React.createElement("tbody",null,
              STATS.map(st=>
                React.createElement("tr",{key:st,style:{borderBottom:`0.5px solid ${G}15`}},
                  React.createElement("td",{style:{padding:"3px 0"}},
                    React.createElement("div",{style:{background:G,borderRadius:2,padding:"1px 5px",display:"inline-block"}},
                      React.createElement("span",{style:{fontSize:10,fontWeight:700,color:"#fff"}},st)
                    )
                  ),
                  React.createElement("td",{style:{padding:"3px 2px"}},
                    React.createElement("input",{value:data[`${st}_v`]??"",onChange:e=>set(`${st}_v`,e.target.value),
                      style:{width:30,textAlign:"center",background:"transparent",border:`0.5px solid ${G}50`,borderRadius:2,color:"var(--color-text-primary)",fontSize:13,fontWeight:600,padding:"1px",outline:"none"}})
                  ),
                  React.createElement("td",{style:{padding:"3px 2px"}},
                    React.createElement("input",{value:data[`${st}_n`]??"",onChange:e=>set(`${st}_n`,e.target.value),
                      style:{width:"100%",background:"transparent",border:"none",borderBottom:`0.5px solid ${G}25`,fontSize:10,color:"var(--color-text-secondary)",padding:"1px 2px",boxSizing:"border-box",outline:"none"}})
                  )
                )
              )
            )
          )
        ),
        // Col 2: Voie du Peuple
        box(null,
          React.createElement("div",null,
            React.createElement("div",{style:{display:"flex",gap:6,alignItems:"center",marginBottom:8,paddingBottom:4,borderBottom:`0.5px solid ${G}40`}},
              React.createElement("span",{style:{fontSize:11,fontWeight:700,color:G,whiteSpace:"nowrap",textTransform:"uppercase",letterSpacing:"0.05em"}},"Voie du Peuple :"),
              React.createElement("input",{value:data.vdp_name??"",onChange:e=>set("vdp_name",e.target.value),placeholder:"nom…",
                style:{flex:1,background:"transparent",border:"none",borderBottom:`0.5px solid ${G}40`,fontSize:11,color:"var(--color-text-primary)",padding:"1px 2px",outline:"none"}})
            ),
            [1,2,3,4,5].map(i=>
              React.createElement("div",{key:i,style:{marginBottom:8,paddingBottom:6,borderBottom:`0.5px solid ${G}20`}},
                React.createElement("div",{style:{display:"flex",alignItems:"center",gap:5,marginBottom:2}},
                  React.createElement("input",{type:"checkbox",checked:!!data[`vdp${i}_c`],onChange:e=>set(`vdp${i}_c`,e.target.checked),style:{accentColor:G,flexShrink:0}}),
                  React.createElement("span",{style:{fontSize:11,color:G,fontWeight:600}},`Rang ${i} :`)
                ),
                React.createElement("input",{value:data[`vdp${i}_t`]??"",onChange:e=>set(`vdp${i}_t`,e.target.value),
                  style:{width:"100%",background:"transparent",border:"none",borderBottom:`0.5px solid ${G}25`,fontSize:11,color:"var(--color-text-primary)",padding:"1px 0",boxSizing:"border-box",outline:"none"}})
              )
            )
          )
        ),
        // Col 3: Stats
        React.createElement("div",null,
          React.createElement("div",{style:{display:"flex",gap:8,marginBottom:8}},
            ...[["init","Init."],["def","Def."]].map(([k,l])=>
              React.createElement("div",{key:k,style:{flex:1}},
                lbl(l),
                React.createElement("input",{value:data[k]??"",onChange:e=>set(k,e.target.value),
                  style:{width:"100%",textAlign:"center",background:"transparent",border:`0.5px solid ${G}50`,borderRadius:2,color:"var(--color-text-primary)",fontSize:15,fontWeight:600,padding:"3px 2px",boxSizing:"border-box",outline:"none"}})
              )
            )
          ),
          box("Pts de Vigueur",
            React.createElement("div",null,
              React.createElement("div",{style:{display:"flex",alignItems:"center",gap:5,marginBottom:2}},
                lbl("Max",{marginBottom:0}),
                React.createElement("input",{value:data.pv_max??"",onChange:e=>set("pv_max",e.target.value),
                  style:{width:38,textAlign:"center",background:"transparent",border:`0.5px solid ${G}40`,borderRadius:2,fontSize:12,padding:"1px",color:"var(--color-text-primary)",outline:"none"}})
              ),
              diamonds("pv")
            ),
            {marginBottom:6}
          ),
          box("Points de Chance",
            React.createElement("div",null,
              React.createElement("div",{style:{fontSize:9,color:G,marginBottom:2}},"+10 à un test"),
              React.createElement("div",{style:{display:"flex",alignItems:"center",gap:5,marginBottom:2}},
                lbl("Max",{marginBottom:0}),
                React.createElement("input",{value:data.pc_max??"",onChange:e=>set("pc_max",e.target.value),
                  style:{width:38,textAlign:"center",background:"transparent",border:`0.5px solid ${G}40`,borderRadius:2,fontSize:12,padding:"1px",color:"var(--color-text-primary)",outline:"none"}})
              ),
              diamonds("pc")
            ),
            {marginBottom:6}
          ),
          box("Dés de Récupération",
            React.createElement("div",null,
              React.createElement("div",{style:{fontSize:9,color:"var(--color-text-secondary)",marginBottom:4}}, "Restauration PV = (d+½ niv.) PV"),
              React.createElement("div",{style:{display:"flex",gap:6}},
                React.createElement("div",null,
                  lbl("Type de dé"),
                  React.createElement("input",{value:data.dr_type??"",onChange:e=>set("dr_type",e.target.value),placeholder:"d8",
                    style:{width:38,background:"transparent",border:`0.5px solid ${G}40`,borderRadius:2,fontSize:12,padding:"2px",color:"var(--color-text-primary)",outline:"none"}})
                ),
                React.createElement("div",null,
                  lbl("Max"),
                  React.createElement("input",{value:data.dr_max??"",onChange:e=>set("dr_max",e.target.value),
                    style:{width:34,textAlign:"center",background:"transparent",border:`0.5px solid ${G}40`,borderRadius:2,fontSize:12,padding:"2px",color:"var(--color-text-primary)",outline:"none"}})
                )
              )
            ),
            {marginBottom:6}
          ),
          box("Points de Mana",
            React.createElement("div",{style:{display:"flex",alignItems:"center",gap:5}},
              lbl("Max",{marginBottom:0}),
              React.createElement("input",{value:data.pm_max??"",onChange:e=>set("pm_max",e.target.value),
                style:{width:52,textAlign:"center",background:"transparent",border:`0.5px solid ${G}40`,borderRadius:2,fontSize:14,fontWeight:600,padding:"2px",color:"var(--color-text-primary)",outline:"none"}})
            )
          )
        )
      ),
      // Identité
      React.createElement("div",{style:{display:"grid",gridTemplateColumns:"1fr 1fr 1fr 1fr",gap:8,marginBottom:10}},
        ...[["famille","Famille"],["profil","Profil"],["ideal","Idéal héroïque"],["travers","Travers"]].map(([k,l])=>
          React.createElement("div",{key:k},
            lbl(l),
            React.createElement("input",{value:data[k]??"",onChange:e=>set(k,e.target.value),
              style:{width:"100%",background:"transparent",border:"none",borderBottom:`0.5px solid ${G}50`,fontSize:12,color:"var(--color-text-primary)",padding:"2px 0",boxSizing:"border-box",outline:"none"}})
          )
        )
      ),
      // Attaques + Armes
      React.createElement("div",{style:{display:"grid",gridTemplateColumns:"215px 1fr",gap:10}},
        box("Attaques",
          React.createElement("div",null,
            React.createElement("div",{style:{display:"grid",gridTemplateColumns:"auto 1fr 36px 36px",gap:4,alignItems:"center",marginBottom:4}},
              React.createElement("div"),
              lbl("Total",{textAlign:"center",marginBottom:0}),
              lbl("Niv.",{textAlign:"center",marginBottom:0}),
              lbl("Mod.",{textAlign:"center",marginBottom:0})
            ),
            ...[["c","CONTACT","FOR"],["d","DISTANCE","AGI"],["m","MAGIQUE","VOL"]].map(([k,attLbl,mod])=>
              React.createElement("div",{key:k,style:{display:"grid",gridTemplateColumns:"auto 1fr 36px 36px",gap:4,alignItems:"center",marginBottom:6}},
                React.createElement("div",{style:{background:G,borderRadius:2,padding:"2px 5px"}},
                  React.createElement("span",{style:{fontSize:9,fontWeight:700,color:"#fff",letterSpacing:"0.03em"}},attLbl)
                ),
                React.createElement("input",{value:data[`att_${k}_t`]??"",onChange:e=>set(`att_${k}_t`,e.target.value),placeholder:"0",
                  style:{textAlign:"center",background:"transparent",border:`0.5px solid ${G}50`,borderRadius:2,fontSize:13,fontWeight:600,padding:"2px",color:"var(--color-text-primary)",outline:"none",width:"100%"}}),
                React.createElement("input",{value:data[`att_${k}_n`]??"",onChange:e=>set(`att_${k}_n`,e.target.value),
                  style:{textAlign:"center",background:"transparent",border:`0.5px solid ${G}30`,borderRadius:2,fontSize:10,padding:"1px",color:"var(--color-text-secondary)",outline:"none",width:"100%"}}),
                React.createElement("input",{value:data[`att_${k}_m`]??"",onChange:e=>set(`att_${k}_m`,e.target.value),placeholder:mod,
                  style:{textAlign:"center",background:"transparent",border:`0.5px solid ${G}30`,borderRadius:2,fontSize:10,padding:"1px",color:"var(--color-text-secondary)",outline:"none",width:"100%"}})
              )
            )
          )
        ),
        React.createElement("div",null,
          box("Armes",
            React.createElement("div",null,
              ...[0,1,2].map(i=>
                React.createElement("div",{key:i,style:{marginBottom:i<2?6:0,paddingBottom:i<2?5:0,borderBottom:i<2?`0.5px solid ${G}20`:"none"}},
                  React.createElement("div",{style:{display:"grid",gridTemplateColumns:"1fr 85px 55px",gap:5,marginBottom:2}},
                    React.createElement("input",{value:data[`arme${i}_n`]??"",onChange:e=>set(`arme${i}_n`,e.target.value),placeholder:"Nom de l'arme",
                      style:{background:"transparent",border:"none",borderBottom:`0.5px solid ${G}40`,fontSize:12,color:"var(--color-text-primary)",padding:"1px 3px",outline:"none"}}),
                    React.createElement("div",{style:{display:"flex",alignItems:"center",gap:3}},
                      React.createElement("span",{style:{fontSize:10,color:G,fontWeight:700,whiteSpace:"nowrap"}},"1d20+"),
                      React.createElement("input",{value:data[`arme${i}_a`]??"",onChange:e=>set(`arme${i}_a`,e.target.value),
                        style:{width:28,textAlign:"center",background:"transparent",border:`0.5px solid ${G}40`,borderRadius:2,fontSize:11,padding:"1px",color:"var(--color-text-primary)",outline:"none"}})
                    ),
                    React.createElement("input",{value:data[`arme${i}_d`]??"",onChange:e=>set(`arme${i}_d`,e.target.value),placeholder:"DM",
                      style:{textAlign:"center",background:"transparent",border:`0.5px solid ${G}40`,borderRadius:2,fontSize:11,padding:"2px",color:"var(--color-text-primary)",outline:"none"}})
                  ),
                  React.createElement("input",{value:data[`arme${i}_s`]??"",onChange:e=>set(`arme${i}_s`,e.target.value),placeholder:"Spécial / portée…",
                    style:{width:"100%",background:"transparent",border:"none",borderBottom:`0.5px solid ${G}15`,fontSize:10,color:"var(--color-text-secondary)",padding:"1px 0",boxSizing:"border-box",fontStyle:"italic",outline:"none"}})
                )
              )
            )
          ),
          React.createElement("div",{style:{marginTop:6}},
            box("Équipement",
              React.createElement("textarea",{value:data.equip??"",onChange:e=>set("equip",e.target.value),placeholder:"Listez votre équipement…",
                style:{width:"100%",background:"transparent",border:"none",fontSize:11,color:"var(--color-text-primary)",resize:"vertical",minHeight:50,boxSizing:"border-box",outline:"none",lineHeight:1.5,padding:0,fontFamily:"var(--font-sans)"}})
            )
          )
        )
      )
    );

  const renderPage2 = () =>
    React.createElement("div",null,
      box("Description du personnage",
        React.createElement("textarea",{value:data.desc??"",onChange:e=>set("desc",e.target.value),placeholder:"Décrivez votre personnage, son histoire, son apparence, sa personnalité…",
          style:{width:"100%",background:"transparent",border:"none",fontSize:12,color:"var(--color-text-primary)",resize:"vertical",minHeight:90,boxSizing:"border-box",outline:"none",lineHeight:1.6,padding:0,fontFamily:"var(--font-sans)"}}),
        {marginBottom:12}
      ),
      React.createElement("div",{style:{display:"grid",gridTemplateColumns:"1fr 1fr 1fr",gap:10}},
        [1,2,3,4,5,6].map(v=>renderVoieCol(v))
      )
    );

  return React.createElement("div",{style:{padding:"0.75rem 1rem",fontFamily:"var(--font-sans)"}},
    // Top bar
    React.createElement("div",{style:{display:"flex",alignItems:"center",gap:8,marginBottom:10,paddingBottom:8,borderBottom:`1px solid ${G}45`}},
      React.createElement("button",{onClick:()=>setCur(null),style:{background:"none",border:`0.5px solid ${G}55`,borderRadius:4,padding:"4px 9px",fontSize:11,color:G,cursor:"pointer",fontWeight:500}}, "← Retour"),
      React.createElement("span",{style:{fontSize:14,fontWeight:600,color:G,flex:1,overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap"}}, cur),
      React.createElement("span",{style:{fontSize:10,color:saved?"var(--color-text-tertiary)":"var(--color-text-secondary)"}}, saved?"✓ sauvegardé":"Sauvegarde…"),
      React.createElement("div",{style:{display:"flex",border:`0.5px solid ${G}50`,borderRadius:6,overflow:"hidden"}},
        ...["Feuille 1","Feuille 2"].map((t,i)=>
          React.createElement("button",{key:i,onClick:()=>setTab(i),style:{padding:"5px 14px",fontSize:12,border:"none",cursor:"pointer",background:tab===i?G:"transparent",color:tab===i?"#fff":G,fontWeight:tab===i?600:400}}, t)
        )
      )
    ),
    tab===0 ? renderPage1() : renderPage2()
  );
}

ReactDOM.createRoot(document.getElementById("root")).render(React.createElement(App));
</script>
