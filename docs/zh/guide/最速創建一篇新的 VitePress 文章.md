# 最速創建一篇新的 VitePress 文章

## 複製粘貼原始md文件
  - 進入您的 VitePress 項目目錄
  - 複製原始 Markdown 文件到 `docs/zh/guide/` 目錄
  - 重命名文件，例如 `最速創建一篇新的 VitePress 文章.md`

![GitHub全自动部署-11.png](https://cloudflare-imgbed-7oz.pages.dev/file/1758984483979_GitHub全自动部署-11.png)

## 添加文章索引
  - 進入您的 VitePress 項目目錄
  - 打開 `docs/zh/config.ts` 文件
  - 搜索 `function sidebarGuide` 函數
  - 在該函數裏的 `items` 數組中，依葫蘆畫瓢填入一下内容
```typescript
{ text: '最速創建一篇新的 VitePress 文章', link: '最速創建一篇新的 VitePress 文章' }
```