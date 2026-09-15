# Demo Page — 部署与维护

论文 demo 页面，静态站点，直接用 GitHub Pages 托管。

## 文件说明

| 文件 | 作用 |
|---|---|
| `index.html` | 页面本体。标题 / 作者 / Abstract / 各 factor 的音频表 + Table 1 指标表 |
| `manifest.js` | **自动生成**，记录每个 setting 对应的 wav 路径。不要手改 |
| `build_manifest.py` | 扫描音频目录，重新生成 `manifest.js` |
| `.nojekyll` | 告诉 GitHub Pages 不要用 Jekyll 处理（否则下划线开头的目录会被忽略） |
| `data_size/` `hop_size/` `Feature_Dimension/` `Codebook size_VQAE/` `RVQ_AE/` | 各 factor 的音频，每个 setting 两条中文 + 两条英文 |
| `origin/` | 原始录音（Ground-truth 行），文件名与上面一致 |

---

## 一、部署到 GitHub Pages

### 1. 建仓库

在 GitHub 上新建一个仓库，例如 `codec-demo`（Public，不要勾 "Add a README"）。

### 2. 本地初始化并推送

在 `MOS_Page` 目录下打开终端（PowerShell 或 Git Bash）：

```bash
git init
git branch -M main
git add .
git commit -m "Add demo page"
git remote add origin https://github.com/RavenCaffeine/codec-demo.git
git push -u origin main
```

> wav 总量大概几十 MB，直接进 git 没问题（GitHub 单文件上限 100 MB，仓库建议 < 1 GB）。
> 如果以后音频涨到几百 MB，再考虑 Git LFS 或把音频放 Release / 对象存储。

### 3. 打开 Pages

仓库页面 → **Settings** → 左侧 **Pages** →
- Source: **Deploy from a branch**
- Branch: **main**，文件夹选 **/ (root)** → **Save**

等 1–2 分钟，页面地址为：

```
https://RavenCaffeine.github.io/codec-demo/
```

（Actions 标签页里能看到 "pages build and deployment" 的进度。）

### 4. 补上论文链接

`index.html` 里搜 `aria-disabled`，把这三个按钮的 `href="#"` 换成真实链接，然后删掉 `aria-disabled="true"`：

```html
<a class="btn" href="https://arxiv.org/abs/XXXX.XXXXX"><span class="dot"></span>Paper (arXiv)</a>
```

---

## 二、后续修改音频

**原则：只动 wav 文件，然后重新生成 manifest，不用改 HTML。**

页面每个 factor 是一张表：行 = setting，列 = 四条固定的 utterance
（中文 `000000000044275` / `000000001596680`，英文 `000000000204281` / `000000001049573`），
最上面一行是 `origin/` 里的原始录音。脚本按 **utterance ID** 对齐，所以文件名必须保持
`zh_<id>.wav` / `en_<id>.wav`。

### 情况 1：替换某个 setting 的音频

把新 wav 丢进对应目录、覆盖同名文件，然后：

```bash
python build_manifest.py
git add . && git commit -m "update audio" && git push
```

脚本会打印每个目录的检查结果（缺文件、空目录、缺 ground truth 都会列出来）。

### 情况 2：换掉这四条示例句子

在所有 setting 目录和 `origin/` 里换成新的 `zh_<新id>.wav` / `en_<新id>.wav`，
重跑脚本即可 —— 列头和 Ground-truth 行会自动跟着变，HTML 不用动。

### 情况 3：增删 setting（比如加一个 `hop16384`）

1. 建目录、放 wav；
2. `build_manifest.py` 里 `FACTORS` 加一行 `("hop16384", "16384")`；
3. `index.html` 里对应 factor 的 `rows` 加一行（`id` 是**目录名**，`label` 是显示文字，
   `m` 是 Table 1 的十个指标）；
4. 重新跑脚本、push。

### 情况 4：改目录名

改了目录名要同步改 `build_manifest.py` 里 `FACTORS` 的第二个字段，以及各 setting 的 `id`
（`id` 必须等于子目录名，和 `index.html` 里的 `rows[].id` 对应）。

建议把带空格的 `Codebook size_VQAE` 改成 `codebook_vqae` 之类 —— URL 里空格要转义，
虽然现在脚本已经帮你转了，但少一个坑。

### 情况 5：改指标数字

指标写在 `index.html` 的 `FACTORS` 数组里，每行的 `m: [...]`，顺序是
`Mel-MAE, MCD, PESQ, F0-RMSE, F0-CORR, WER, PER, SPK-SIM, UT-MOS, CMOS`。

---

## 三、本地预览

不要直接双击 `index.html`（部分浏览器对 `file://` 下的音频有限制）。用：

```bash
python -m http.server 8000
```

然后浏览器打开 `http://localhost:8000`。
