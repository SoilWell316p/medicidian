```dataviewjs
dv.list(
dv.pages(`[[${this.currentFilePath}]]`)
	.map(page => page.file.link))
```

``` dataviewjs
const files = dv.pages().file.path;

if (files.length < 2) {
    dv.paragraph("Vault内に十分なファイルがありません。");
} else {
    // ファイルをランダムにシャッフル
    const shuffledFiles = files.sort(() => Math.random() - 0.5);

    // ランダムな異なる2つのファイルを取得
    const randomFiles = shuffledFiles.slice(0, 2);

    // リストとして表示
    dv.list(randomFiles.map(file => `[${file}](${file})`));
}

```


