## 背景

`baidubce/blogs` 作为 Qianfan research / blog / tech report / project page 的统一展示入口。
本次 PR 采用最小静态页面方案，在 `qianfan/` 下提供聚合页，并接入 Prefix Sampling 与 Qianfan-VL。

## 修改内容

- 新增 Qianfan 聚合页
  - `qianfan/index.html`
  - `qianfan/index_en.html`
- 新增 Prefix Sampling 博客页面
  - 英文主页面：`qianfan/prefix-sampling/index.html`
  - 中文页面：`qianfan/prefix-sampling/index_zh.html`
- 上传 Prefix Sampling 相关图片资源
- 聚合页加入已有项目入口
  - Prefix Sampling
  - Qianfan-VL
  - Qianfan-OCR
- 统一中英文页面内容与样式
  - 首页条目
  - 博客标题区
  - TOC 样式
  - 页脚文案
- 更新 Prefix Sampling citation URL
  - `https://baidubce.github.io/blogs/qianfan/prefix-sampling/`

## URL 规则

- 聚合页英文：`https://baidubce.github.io/blogs/qianfan/`
- 聚合页中文：`https://baidubce.github.io/blogs/qianfan/index.html`
- Prefix Sampling 英文：`https://baidubce.github.io/blogs/qianfan/prefix-sampling/`
- Prefix Sampling 中文：`https://baidubce.github.io/blogs/qianfan/prefix-sampling/index_zh.html`

## 后续接入方式

后续新项目可继续按相同方式接入：

- 在 `qianfan/` 聚合页增加条目
- 在 `qianfan/<project>/` 下放置对应静态页面与资源

这样可以保持结构清晰，维护成本也较低。
