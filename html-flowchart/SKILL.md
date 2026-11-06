---
name: html-flowchart
description: 画流程图时默认用 HTML 作图，并按手机屏幕排版。用户说 画流程图 / 流程图 / 业务流程图 / 审批流程 / 画个图 时使用，用户不必每次重申"用 HTML"。
compatibility: 无要求，产出是一个 html 代码块。
---

# 流程图用 HTML

要画流程图，输出一段自包含的 ```html 代码块：`<style>` 内联，不用 JS、不用 CDN、不用 SVG 坐标，
不要 ASCII 方框图，不要 Mermaid 代码块。

按手机屏幕排版：必须有 `<meta name="viewport" content="width=device-width,initial-scale=1">`，
单列竖排为主，宽度用百分比加 `max-width`，不要出现横向滚动，字号别小于 14px。

代码块外只补一句"手机浏览器打开即可"。
