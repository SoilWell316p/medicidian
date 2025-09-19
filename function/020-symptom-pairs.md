---
symptom1: "[[c060000-sym-発熱]]"
symptom2: "[[c000000-sym-上肢痛]]"
---

```dataviewjs

const key1 = dv.current().symptom1.path
const key2 = dv.current().symptom2.path 


const syms = dv.pages().filter(p => p.file.path.includes("-sym-"));


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


function intersection(arr1, arr2) {
	let commonElem = arr1.filter((valA) => arr2.some((valB) => valB.path === valA.path));
	return commonElem;
};


if (syms.length < 2) {
	dv.paragraph("not enough files in your vault!");
} else {
	//const shuffledFiles = syms.sort(() => Math.random() - 0.5);
	//const randomFiles = shuffledFiles.slice(0, 2);
	//let file1 = randomFiles[0];
	//let file2 = randomFiles[1]; 

	let file1 = syms.filter(p => p.file.path.match(key1))[0].file;
	let file2 = syms.filter(p => p.file.path.match(key2))[0].file;
	
	dv.paragraph(`symptom#1: ${file1.link}`);
	dv.paragraph(`symptom#2: ${file2.link}`);

	let links1 = makeLinks(file1);
	let links2 = makeLinks(file2);

	let common = intersection(links1, links2);
	if (common.length < 1) {
		dv.paragraph("no common elements");
	} else {
		dv.paragraph("common diagnoses...");
		dv.paragraph(`total number: ${common.length - 1}`)
		dv.list(common);
	};
};

```


