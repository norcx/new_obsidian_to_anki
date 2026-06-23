# better-obsidian-to-anki

[English documentation](README.md)

better-obsidian-to-anki fork 自 [Obsidian_to_Anki](https://github.com/Pseudonium/Obsidian_to_Anki)，保留原插件“从 Obsidian Markdown 批量同步卡片到 Anki”的核心能力，并在个人使用场景上增加了路径 deck、block 级回跳、context 展示和数学/代码格式化优化。

原插件的基础笔记语法、AnkiConnect 配置、自定义正则、图片/音频同步、Frozen Fields、删除标记和扫描流程仍然适用。本文重点说明本 fork 相比上游新增或调整的行为。

## 功能总览

### 1. 按 Obsidian 文件路径同步到 Anki deck

开启 `Use Path as Deck` 后，插件会把 Markdown 文件路径转换成 Anki deck 路径。

例如：

```text
Obsidian 文件: root/hello/world.md
Anki deck:    root::hello::world
```

这样可以让 Anki deck 结构和 Obsidian 文件夹结构保持一致，适合按课程、章节、项目或学科目录管理卡片。

新生成的配置中，`Use Path as Deck` 默认开启。

如果设置了 `Scan Directory`，生成 deck 名称时会先移除这个扫描目录前缀。例如：

```text
Scan Directory: anki
Obsidian 文件: anki/LinearAlgebra/矩阵求导.md
Anki deck:    LinearAlgebra::矩阵求导
```

注意：开启后会优先使用文件路径生成 deck。如果你仍希望在单个文件中通过 `TARGET DECK` 指定 deck，需要关闭 `Use Path as Deck`。

如果你移动或重命名 Markdown 文件，下次扫描会把这个新路径视为已变更文件，并把已有 Anki 卡片移动到新的路径 deck。

### 2. 使用 Obsidian block id 作为卡片 ID

原插件默认写入的卡片 ID 类似：

```text
ID: 1714433449297
```

本 fork 可以写成 Obsidian block id 形式：

```text
^ID-1714433449297
```

这样 Obsidian 能把 `^ID-...` 当作 block 锚点识别，Anki 卡片上的来源链接可以直接跳回对应 Markdown block，而不是只打开文件。

![Path deck and source link demo](Images/demo.gif)

### 3. 从 Anki 跳回 Obsidian block

开启 `Add File Link` 和 `Add Card link` 后，插件会在指定字段中追加一个 Obsidian 来源链接。这个链接会指向当前卡片对应的 block ID。

生成的链接概念上类似：

```html
<a href="obsidian://open?...#^ID-1714433449297" class="obsidian-link">link</a>
```

这个功能依赖两个条件：

1. Markdown 卡片已经有，或即将写入，`^ID-...` block id。
2. 本机 Obsidian 能正常打开 `obsidian://open` 链接。

新增卡片第一次同步时，最终 Anki note id 要等 AnkiConnect 创建卡片后才知道。因此插件会先添加卡片，拿到 note id，把 block id 写回 Markdown，再执行一次字段更新，让来源链接指向真实 block。

### 4. Context 展示优化

开启 `Add Context` 后，插件会把卡片所在位置的文件和标题上下文写入指定字段。相比上游，本 fork 做了这些调整：

1. context 放在卡片正文前面，而不是追加到末尾。
2. 文件路径和标题层级使用换行展示。
3. context 与正文之间插入 `<hr>` 分隔线。
4. 标题中的 Markdown 链接会转换成 HTML 链接。
5. 标题中的 `$...$` 公式会转换成 Anki 可识别的行内数学格式。

输出大致如下：

```html
<br>课程/第五章.md<br>集成计数器<br>同步二进制计数器<br><hr>
```

这个功能适合把卡片放回原始知识结构中复习，尤其适合长文档、章节笔记和课程笔记。

### 5. 数学公式和代码格式化优化

本 fork 调整了 Markdown 转 Anki HTML 的处理顺序：

1. 先保护代码块和行内代码。
2. 再处理 Obsidian 数学公式。
3. 再处理 cloze、高亮、图片、音频和链接。
4. 最后恢复被保护的代码内容。
5. 再把剩余 Markdown 转成 HTML。

这样可以避免代码中的 `$` 被误识别成 LaTeX。转义美元符号 `\$` 不会作为公式开始或结束，并会在最终输出中还原为 `$`。

### 6. 链接行为

笔记正文中的普通 Obsidian 链接仍然会转换为 `obsidian://open` HTML 链接。

指向 Obsidian block 的 Markdown 链接也会按同样方式处理：格式化后仍然作为 Obsidian 链接保留。

## 安装和使用

### 前置条件

1. 已安装 Obsidian。
2. 已安装 Anki。
3. 已安装并启用 AnkiConnect。
4. AnkiConnect 允许 Obsidian 调用，常见配置需要包含 `app://obsidian.md`。

完整的上游安装说明可以参考：
[Obsidian_to_Anki](https://github.com/Pseudonium/Obsidian_to_Anki)

### 手动构建和安装

安装依赖：

```bash
npm install
```

构建插件：

```bash
npm run build
```

把以下文件放入 Obsidian 插件目录，替换原 `obsidian-to-anki-plugin` 的对应文件：

```text
main.js
manifest.json
styles.css
```

然后重启 Obsidian，并启用插件。

### 推荐设置

| 设置项 | 建议 | 作用 |
| --- | --- | --- |
| `Add File Link` | 开启 | 在 Anki 字段里加入回到 Obsidian 的链接。 |
| `Add Card link` | 开启 | 让来源链接定位到具体 Obsidian block。 |
| `Use Path as Deck` | 默认开启 | 将 Markdown 文件路径映射为 Anki deck 路径；设置 `Scan Directory` 时会移除扫描目录前缀。 |
| `Add Context` | 开启 | 在卡片字段前面加入路径和标题上下文。 |
| `ID Comments` | 使用 block id 时通常关闭 | HTML 注释会影响 Obsidian 识别 block id。 |

## 自定义正则兼容 block id

如果你的自定义正则以前使用下面的判断来避免匹配 ID 行：

```regex
(?<!<!--)
```

使用 block id 后建议改成：

```regex
(?<!<!--)(?<!\^ID-)
```

原因是 block id 行现在形如：

```text
^ID-1714433449297
```

如果正则没有排除 `^ID-`，可能会把 ID 行误当作下一张卡片的正文。

## 填空题建议

填空题建议使用 Anki 原生 cloze 格式：

```text
{{c1::需要挖空的内容}}
```

这样可以减少误匹配 BookxNote 链接、LaTeX 片段或其他 Markdown 内容的概率。

一个可用的自定义正则示例：

```regex
((?:.+\n)*(?:.*{{c.*)(?:\n(?:^.{1,3}$|^.{4}(?<!<!--)(?<!\^ID-).*))*)
```

## 开发

运行 TypeScript/Rollup 构建：

```bash
npm run build
```

运行上游测试套件：

```bash
npm test
```

完整测试依赖上游 Docker/Webdriver 配置和可用的 Anki 测试环境。

## 备注

本项目仍沿用上游插件架构。主要改动集中在：

```text
src/file.ts
src/files-manager.ts
src/format.ts
src/note.ts
src/settings.ts
src/setting-to-data.ts
src/interfaces/settings-interface.ts
```

已知边界问题和清理任务先保留，后续可以通过单独的 issue 或 PR 处理。
