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
const phy_files = dv.pages('#physical').map(p => p.file).array();


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


// utilities
function editDict (links, dict) {
	let paths = links.map(link => link.path);
	for (let i = 0; i < paths.length; i++) {
		dict[paths[i]] = links[i];
	};
};

function makeLinks (file) {
	let outlinks = file.outlinks.filter(link => dv.page(link)).filter(link => !link.path.includes("-moc-"));
	let inlinks = file.inlinks.filter(link => dv.page(link)).filter(link => !link.path.includes("-moc-"));
	let path2link = {};
	editDict(outlinks, path2link);
	editDict(inlinks, path2link);
	let uniqueLinks = Object.values(path2link);
	return uniqueLinks;
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
  
  // calculate ddx when CHECK is pressed
  b2.onclick = () => {
    try {
      const page = dv.page(target.path);
      const ddx_list = makeLinks(page.file);
      dv.paragraph("他に挙げられる鑑別は、、、");
      dv.list(ddx_list);
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

