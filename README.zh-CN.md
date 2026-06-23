# better-obsidian-to-anki

[English documentation](README.md)

better-obsidian-to-anki fork 自 [Obsidian_to_Anki](https://github.com/Pseudonium/Obsidian_to_Anki)。原项目已经很久没有活跃维护，所以这个 fork 保留原来的 Obsidian 到 Anki 同步流程，并加入一些我自己学习时希望有的功能。

最主要的新增功能是来源跳转：Anki 里的卡片可以跳回生成它的 Obsidian block。这个 fork 也支持按 Obsidian 文件路径组织 Anki 牌组，例如 `LinearAlgebra/matrix-derivatives.md` 可以同步到 `LinearAlgebra::matrix-derivatives`。

![Path deck and source link demo](Images/demo.gif)

## 改动概览

- Anki 卡片可以通过 `obsidian://open` 链接跳回具体 Obsidian block。
- 卡片 ID 可以写成 Obsidian block id，例如 `^ID-1714433449297`。
- 开启 `Use Path as Deck` 后，Anki 牌组可以跟随 Obsidian 文件路径。
- 设置了 `Scan Directory` 时，生成路径牌组会移除扫描目录前缀。
- 移动或重命名 Markdown 文件后，可以把已有 Anki 卡片移动到新的路径牌组。
- context 可以放在字段开头，显示文件路径和标题层级。
- 数学公式和代码块的格式化顺序更安全，代码里的 `$` 不会被误识别为 LaTeX。

原插件文档仍然适用于基础笔记语法、AnkiConnect 配置、自定义正则、媒体同步、Frozen Fields 和删除标记。

## 路径牌组

开启 `Use Path as Deck` 后：

```text
Obsidian file: root/hello/world.md
Anki deck:    root::hello::world
```

如果设置了 `Scan Directory`，会移除这个前缀：

```text
Scan Directory: anki
Obsidian file:  anki/LinearAlgebra/matrix-derivatives.md
Anki deck:      LinearAlgebra::matrix-derivatives
```

新生成配置中，`Use Path as Deck` 默认开启。移动或重命名 Markdown 文件后，下次扫描会把已有 Anki 卡片移动到新的路径牌组。

## Block 跳转

原插件写入的 note ID 类似：

```text
ID: 1714433449297
```

这个 fork 可以写成 Obsidian block id：

```text
^ID-1714433449297
```

开启 `Add File Link` 和 `Add Card link` 后，插件会在指定 Anki 字段里追加来源链接。这个链接指向具体 block：

```html
<a href="obsidian://open?...#^ID-1714433449297" class="obsidian-link">link</a>
```

新增卡片时，插件会先创建 Anki note，拿到 note ID，把 block ID 写回 Markdown，然后再次更新 Anki 字段，让链接指向真实 block。

## Context 和格式化

开启 `Add Context` 后，文件和标题上下文会放在卡片内容前面，并用换行和分割线组织：

```html
<br>course/chapter-5.md<br>Integrated Counters<br>Synchronous Counters<br><hr>
```

Markdown 格式化时会先保护代码块，再处理数学公式、cloze、高亮、媒体和链接。转义美元符号 `\$` 会在最终输出中还原为 `$`。

## 安装

需要：

- Obsidian
- Anki
- AnkiConnect
- AnkiConnect 允许 Obsidian 调用，通常需要包含 `app://obsidian.md`

手动构建：

```bash
npm install
npm run build
```

把这些文件复制到 Obsidian 插件目录：

```text
main.js
manifest.json
styles.css
```

## 推荐设置

| 设置项 | 建议 | 作用 |
| --- | --- | --- |
| `Add File Link` | 开启 | 在 Anki 字段里加入回到 Obsidian 的链接。 |
| `Add Card link` | 开启 | 让来源链接定位到具体 Obsidian block。 |
| `Use Path as Deck` | 默认开启 | 将 Markdown 路径映射为 Anki 牌组路径。 |
| `Add Context` | 按需开启 | 在卡片内容前加入文件和标题上下文。 |
| `ID Comments` | 使用 block id 时通常关闭 | HTML 注释可能影响 Obsidian 识别 block id。 |

## 自定义正则兼容

如果你的自定义正则以前用这个条件避免匹配 ID 行：

```regex
(?<!<!--)
```

使用 block id 后建议改成：

```regex
(?<!<!--)(?<!\^ID-)
```

否则 `^ID-...` 行可能被误当作卡片正文。

## 开发

```bash
npm run build
npm test
```

完整测试依赖上游 Docker/Webdriver 配置和可用的 Anki 测试环境。
