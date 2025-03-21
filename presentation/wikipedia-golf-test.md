### wikipedia golf test
- **ランダムに2つのファイルを選択し、これらのファイルをつなぐ最短の経路を探索する。**

```dataviewjs
// find connected components

const files = dv.pages()
    .filter(p => p.file.folder.match(/^c\d{2}-.+/))
    .filter(p => !p.file.name.includes("-moc-"))
    .map(p => p.file);

// initialization
let graph = new Map();
let visited = new Set();
let components = [];
let nodeToComponent = new Map();

// make a graph
files.forEach(f => {
	let outlinks = f.outlinks.filter(link => dv.page(link)).filter(link => !link.path.includes("-moc-"));
	let inlinks = f.inlinks.filter(link => dv.page(link)).filter(link => !link.path.includes("-moc-"));
	
	let links = [
	...(outlinks.map(l => l.path)),
	...(inlinks.map(l => l.path))
	];
	let uniqueLinks = Array.from(new Set(links));
	graph.set(f.path, uniqueLinks);
});

// depth first search
function dfs(node, group) {
	if (visited.has(node)) return;
	visited.add(node);
	group.push(node);
	nodeToComponent.set(node, components.length);
	graph.get(node).forEach(neighbor => dfs(neighbor, group));
}

// apply dfs to each node
files.forEach(f => {
	if (!visited.has(f.path)) {
		let group = [];
		dfs(f.path, group);
		components.push(group);
	}
});


// broadth first search
function shortestPath(graph, start, goal) {
    if (start === goal) return { steps: 0, path: [start] };

    let queue = [[start, 0, [start]]]; 
    let visitedNodes = new Set([start]);

    while (queue.length > 0) {
        let [current, steps, path] = queue.shift();

        let neighbors = graph.get(current) || []; 

        for (let neighbor of neighbors) {
            if (neighbor === goal) {
                return { steps: steps + 1, path: [...path, neighbor] };
            }
            if (!visitedNodes.has(neighbor)) {
                visitedNodes.add(neighbor);
                queue.push([neighbor, steps + 1, [...path, neighbor]]);
            }
        }
    }
    return { steps: -1, path: [] }; 
}

function path2link(path) {
	let page = dv.pages().filter(p => p.file.path.startsWith(path));
	return page.file.link;
}

// random file selection
if (files.length < 2) {
	dv.paragraph("not enough files in your vault!");
} else {
	const shuffledFilepaths = files.path.sort(() => Math.random() - 0.5);
	const randomFilepaths = shuffledFilepaths.slice(0, 2);
	let filepath1 = randomFilepaths[0];
	let filepath2 = randomFilepaths[1];

	dv.paragraph(`selected files: \n start: ${filepath1}, \n goal: ${filepath2}`);
	let component1 = nodeToComponent.get(filepath1);
	let component2 = nodeToComponent.get(filepath2);
	dv.paragraph(`component#: \n ${component1}, ${component2}`)

	// calculation
	if (component1 === component2) {
		let pathway = shortestPath(graph, filepath1, filepath2);
		dv.paragraph(`shortest step: ${pathway.steps}`);
		dv.paragraph(`path example: \n`);
		for (let node of pathway.path) {
			dv.paragraph(path2link(node));
			if (node !== filepath2) {
				dv.paragraph("↓")
				} 
			}
	} else {
		dv.paragraph(`belong to different components`);
	}
}
```

