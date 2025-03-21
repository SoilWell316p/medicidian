## Personal Knowledge Management
こんにちは。hinaと申します。現在、臨床実習真っただ中の医学部生です。実習や国家試験の勉強をしていると「学んだことをノートにまとめたい」という衝動に駆られることがよくあります。というわけで「いかに効率的にまとめるか、アウトプットするか」について僕が学んできたことを共有させていただこうと思います。
皆さんは、授業や実習で習ったことをどうやってまとめているでしょうか？まとめノートを作ったり、レジュメを使っているひともいると思います。こうした「知識の管理方法」はPersonal Knowledge Management (PKM)と呼ばれ、情報工学や経営学の研究対象となるほど人々の関心を集めてきた分野なのです。
PKMのためのデジタルツールとしてさまざまなソフトウェアが開発されています。メジャーどころでは[Notion](https://www.notion.com/ja)や[ScrapBox](https://scrapbox.io/)、論文管理・研究目的であれば[Zotero](https://www.zotero.org/)など、使われている方もいるのではないでしょうか。今回は、[Obsidian](https://obsidian.md/)というソフトウェアを使った国家試験対策について紹介させていただこうと思います。

## Obsidianによる医学知識の管理
Obsidianとは、PKMに特化したローカルメモアプリです。ObsidianはPKMの中でも特に[Zettelkasten](https://ja.wikipedia.org/wiki/%E3%83%84%E3%82%A7%E3%83%83%E3%83%86%E3%83%AB%E3%82%AB%E3%82%B9%E3%83%86%E3%83%B3)と呼ばれる手法と親和性が高いと言われています。Zettelkastenとは社会学者のニクラス・ルーマンによって提唱された情報管理システムであり、①カード（紙、電子メモなど）と、それに保存される情報として②ID・タグ、③他のカードへのリンク、の3つの要素から構成されています。いわゆる「マインドマップ」に近いメモ作成術だと思っています。
Zettelkastenの大きな特徴は、知識の階層ではなく、繫がりを重視するところです。これは医学の勉強を進めるうえで大きなメリットになると思います。たとえば、「脳卒中」についてまとめるとします。ここで問題になるのが、脳卒中は脳梗塞、脳出血、SAHに細分されるし、脳梗塞は梗塞部位別にさらに細分することができる、、、というように、果てしなく階層づけが出来てしまうのです。こんなものをいちいちフォルダ毎にまとめてしまうときりがありません。そこでZettelkastenでは、知識を階層づけするのではなく、ひとつひとつのメモを平等に扱います。たとえば[LYT](https://goryugo.com/20201210/obsidian-lyt/)というフレームワークではサブフォルダを作成する代わりに、1つのMOCファイル（Map Of Contents）を作成し、そこに他のファイルへのリンクをどっさり乗っけてしまうのです。このMOCファイルが目次のような役割を果たしつつ、ファイル間を行き来するハブとして機能してくれるというわけです。
というわけで、かれこれ半年ほどObsidianを使い続け、とうとうファイル数が1000を突破しました（びっくり）。Obsidianの面白機能に、ファイルとそのリンク関係をグラフとして可視化してくれるものがあります。1000枚のファイルが星座のように繫がっていく様子は、見ていてやみつきになるところがあります。

## Wikipediaゴルフと最短経路問題
Obsidianでは、ファイルの中に別ファイルへのリンクを作成することが出来ます。最終的にオリジナルのwikipediaができるわけです。そういえば、みなさんwikipediaで[ゴルフ](https://ja.wikipedia.org/wiki/Wikipedia:Wikipedia%E3%82%B4%E3%83%AB%E3%83%95)が出来るのはご存じでしょうか（唐突）。あるファイルから、別のファイルまで、リンクを踏んで何ステップで到達できるか競うというものです。今回は、Obsidianのプラグイン機能を使って"Wikipediaゴルフ"を実装してみました。

話はガラッと変わって競技プログラミングについて語ります。

```js
// find connected components
dv.paragraph("##### task 1: find connected components #####")

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

// test 1
const num = Math.floor(Math.random() * 300)

let filepath = files[num].path
let nodes = graph.get(filepath)
dv.paragraph(`selected file: ${filepath}`)
dv.paragraph(`linked files: \n ${nodes}`)
dv.paragraph("task 1 completed")
dv.paragraph("---")

dv.paragraph("##### task 2: group files #####")

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

// test 2
dv.paragraph(`number of components: ${components.length}`);
dv.paragraph("task 2 completed")
dv.paragraph("---")

dv.paragraph("##### task 3: find shortest steps (bfs) #####")

// broadth first search
function shortestStep(graph, start, goal) {
	if (start === goal) return 0;

	let queue = [[start, 0]];
	let visitedNodes = new Set([start]);

	while (queue.length > 0) {
		let [current, steps] = queue.shift();
		for (let neighbor of (graph.get(current))) {
			if (neighbor === goal) return steps + 1;
			if (!visitedNodes.has(neighbor)) {
				visitedNodes.add(neighbor);
				queue.push([neighbor, steps + 1]);
			}
		}
	}
	return -1;
}

// random file selection
if (files.length < 2) {
	dv.paragraph("not enough files in your vault!");
} else {
	const shuffledFilepaths = files.path.sort(() => Math.random() - 0.5);
	const randomFilepaths = shuffledFilepaths.slice(0, 2);
	let filepath1 = randomFilepaths[0];
	let filepath2 = randomFilepaths[1];

	dv.paragraph(`selected files: \n ${filepath1}, \n ${filepath2}`);
	let component1 = nodeToComponent.get(filepath1);
	let component2 = nodeToComponent.get(filepath2);
	dv.paragraph(`component#: \n ${component1}, ${component2}`)

	// calculation
	if (component1 === component2) {
		let pathLength = shortestStep(graph, filepath1, filepath2);
		dv.paragraph(`shortest path: ${pathLength}`);
	} else {
		dv.paragraph(`belong to different components`);
	}
}

dv.paragraph("task 3 completed")
dv.paragraph("---")

dv.paragraph("##### task 4: print shortest path #####")

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


## これからやること
#### 1. データベースの整理
#### 2. 新しい機能の実装

## 参考文献
1. [Obsidianを医学の勉強に使う方法（しおん）](https://note.com/shion_medical/n/nc8dc9c359372)
2. [Obsidianで医学知識を管理する（Phenol）](https://note.com/phenomed/n/n57445c7c4aef#wgB4M)
3. [ゼロからはじめるObsidian案内](https://qiita.com/hann-solo/items/22bcaa81b695ddb47238)
4. [ObsidianでPKMを続けられた理由を振り返る](https://minerva.mamansoft.net/%F0%9F%93%98Articles/%F0%9F%93%98Obsidian%E3%81%A7PKM%E3%82%92%E7%B6%9A%E3%81%91%E3%82%89%E3%82%8C%E3%81%9F%E7%90%86%E7%94%B1%E3%82%92%E6%8C%AF%E3%82%8A%E8%BF%94%E3%82%8B)
5. [効率的に成長するためのデジタルノート術 (Obsidian x Zettelkasten/LYT framework)](https://qiita.com/YUM_3/items/a5ad1aa45f7463ee4218)
6. アルゴリズムとデータ構造（けんちょん）
