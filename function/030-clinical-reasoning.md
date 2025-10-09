## 主訴・現病歴
```dataviewjs
// container
const container = dv.el("div");


// get md files in vault
// sym_file: symptoms 
// dis_file: diseases
// phy_file: physical assessment
const sym_files = app.vault.getMarkdownFiles().filter(f => f.path.includes("-sym-"));
const dis_files = dv.pages('#disease').map(p => p.file).array();
const phy_files = dv.pages('#physical or #history').map(p => p.file).array();


// make datalist
// for symptoms
const fileListSym = dv.el("datalist");
fileListSym.id = "vault-files-sym";
sym_files.forEach(f => {
  const opt = document.createElement("option");
  opt.value = f.path;             
  opt.label = f.basename;         
  fileListSym.appendChild(opt);
});
// for diseases
const fileListDis = dv.el("datalist");
fileListDis.id = "vault-files-dis";
dis_files.forEach(f => {
  const opt = document.createElement("option");
  opt.value = f.path;             
  opt.label = f.basename;         
  fileListDis.appendChild(opt);
});
// for physical
const fileListPhy = dv.el("datalist");
fileListPhy.id = "vault-files-phy";
phy_files.forEach(f => {
  const opt = document.createElement("option");
  opt.value = f.path;             
  opt.label = f.basename;         
  fileListPhy.appendChild(opt);
});


// utilities
function editDict (links, dict) {
	let paths = links.map(link => link.path);
	for (let i = 0; i < paths.length; i++) {
		dict[paths[i]] = links[i];
	};
};

function makeLinks (file) {
	const isReal = (l) => dv.page(l);
	const notMoc = (l) => !l.path?.includes("-moc-");
	const hasDisease = (l) => {
		const p = dv.page(l);
		if (!p) return false;
		const tags = (p.file?.tags ?? []).map(t => t.startsWith('#') ? t.slice(1) : t); 
		const etags = (p.file?.etags ?? []);
		return tags.includes('disease') || etags.includes('#disease');
	};
	
	let outlinks = file.outlinks.filter(isReal).filter(notMoc).filter(hasDisease);
	let inlinks = file.inlinks.filter(isReal).filter(notMoc).filter(hasDisease);
	
	let path2link = {};
	editDict(outlinks, path2link);
	editDict(inlinks, path2link);
	
	return Object.values(path2link);
};


// first input: select symptoms
const input1 = dv.el("input");
input1.setAttr("list", "vault-files-sym");
input1.placeholder = "主訴を選択してください";
input1.style = "font-size: large; margin-right: 10px; width: 60%;";
// retrieve the selected symptom page info

//await app.workspace.getLeaf(true).openFile(target);

// start button
const b1 = dv.el("button", "START");
b1.style = "color:white; background: darkred;";
// click move
b1.onclick = async () => {
  // second input: differential diagnosis
  dv.paragraph("---");
  dv.paragraph("## 鑑別疾患");
  dv.paragraph("鑑別疾患をあげてください: ");
  // button for the next step
  const val = input1.value.trim();
  const target = sym_files.find(f => f.path === val);
  if (!target) {
    new Notice("couldn't find the file");
    return;
  };
  // generate UI first
  const b2 = dv.el("button", "CHECK")
  b2.style = "color:white; background: darkred;";
  const cont = dv.el("div")
  // list up differential diagnosis  
  function addRow() {
    const row = dv.el("div"); 
    row.style = "margin-bottom: 5px;"
    const a = dv.el("input");
    a.style = "font-size: large; margin-right: 10px; width: 80%;"
    a.setAttr("list", "vault-files-dis");
    a.placeholder = "疾患を選択してください";
    const b = dv.el("button", "ADD");
    b.style = "color:white; background: darkblue;"
    // add a new row in a recursive way
    b.onclick = addRow;
    row.appendChild(a);
    row.appendChild(b);
    cont.appendChild(row);
  };
  addRow();
  addRow();
 
  container.appendChild(fileListDis);
  
  // calculate ddx when CHECK is pressed
  b2.onclick = () => {
    try {
      const inputs = Array.from(
        cont.querySelectorAll('input[list="vault-files-dis"]')
      );
      const rawVals = inputs.map(i => i.value.trim()).filter(v => v);
      
      
      const page = dv.page(target.path);
      const ddx_list = makeLinks(page.file);
      const ddx_paths = ddx_list.map(l => l.path);
      
      // compare sets 
      const setAns = new Set(rawVals);
      const setDdx = new Set(ddx_paths);
      
      const correct = rawVals.filter(p => setDdx.has(p));
      const over = rawVals.filter(p => !setDdx.has(p));
      const missing = ddx_paths.filter(p => !setAns.has(p));
      
      const base = (p) => p.split("/").pop().replace(/\.md$/i, "");
      const toWiki = (p) => `[[${p}|${base(p)}]]`;
      
      const fmt = {
        correct: correct.map(p => `⭕️ ${toWiki(p)}`),
        over:    over.map(p    => `➕ ${toWiki(p)}`),
        missing: missing.map(p => `➖ ${toWiki(p)}`)
      };

      const score = correct.length - over.length - missing.length;
      
      dv.paragraph("### 判定：");
      if (fmt.correct.length) { dv.paragraph("**correct**"); dv.list(fmt.correct); }
      if (fmt.over.length)    { dv.paragraph("**over**");    dv.list(fmt.over); }
      if (fmt.missing.length) { dv.paragraph("**missing**"); dv.list(fmt.missing); }

      dv.paragraph(`**Score:** ${score}  （⭕️ = +1, ➖/➕ = -1）`);
      
      
      // arrange UI
      const b3 = dv.el("button", "NEXT");
      b3.style = "color:white; background: darkred;";
      b3.onclick = () => {
        dv.paragraph("---");
        dv.paragraph("## 問診と身体診察");
        dv.paragraph("必要な問診項目・身体診察をあげてください：");
        
        const cont1 = dv.el("div");
        // list up necessary physical assessment
        function addRow1() {
          const row = dv.el("div");
          row.style = "margin-bottom: 5px;"
          const a1 = dv.el("input");
          a1.style = "font-size: large; margin-right: 10px; width: 40%;"
          a1.setAttr("list", "vault-files-phy");
          a1.placeholder = "診察項目を選択してください";
          const a2 = dv.el("input");
          a2.style = "font-size: large; margin-right: 10px; width: 40%;"
          a2.setAttr("list", "vault-files-dis");
          a2.placeholder = "想定疾患は？";
          const b = dv.el("button", "ADD");
          b.style = "color:white; background: darkblue;"
          b.onclick = addRow1;
          row.appendChild(a1);
          row.appendChild(a2);
          row.appendChild(b);
          cont1.appendChild(row);
        };
        addRow1();
        addRow1();         
        
      };
      
    } catch (e) {
      console.error(e);
      new Notice("DDX retirieve error");
    };
  };
};


// arrange UI
container.appendChild(input1);
container.appendChild(b1);
container.appendChild(fileListSym);

```

