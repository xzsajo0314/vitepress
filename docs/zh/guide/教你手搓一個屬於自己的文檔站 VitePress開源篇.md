# 教你手搓一個屬於自己的文檔站 VitePress開源篇（實現基於GitHub Actions的全自動部署VitePress）

> 更新时间：2025年9月28日 01点15分

## Fork 本項目到自己的 GitHub 倉庫（文檔站原石）
<https://github.com/vuejs/vitepress>

![GitHub全自动部署-8.png](https://cloudflare-imgbed-7oz.pages.dev/file/1758978709873_GitHub全自动部署-8.png)

![GitHub全自动部署-2.png](https://cloudflare-imgbed-7oz.pages.dev/file/1758920815866_GitHub全自动部署-2.png)

![GitHub全自动部署-3.png](https://cloudflare-imgbed-7oz.pages.dev/file/1758920816732_GitHub全自动部署-3.png)

![GitHub全自动部署-4.png](https://cloudflare-imgbed-7oz.pages.dev/file/1758920819579_GitHub全自动部署-4.png)

![GitHub全自动部署-9.png](https://cloudflare-imgbed-7oz.pages.dev/file/1758979030584_GitHub全自动部署-9.png)

## 在本地創建 SSH 密鑰對（鑰匙、鎖）
   - 打開終端或命令提示符
   - 執行以下命令創建密鑰對
     ```bash
     ssh-keygen -t rsa -b 4096
     ```
   - 將生成的公鑰（鎖） `id_rsa_vitepress.pub` 內容複製到 GitHub 倉庫的部署密鑰設置中
     - 執行以下命令查看公鑰內容
     ```bash
     cat ~/.ssh/id_rsa_vitepress.pub
     ```
     或者
     ```bash
     type $env:USERPROFILE\.ssh\id_rsa_vitepress.pub
     ```
     - 複製終端上的公鑰內容
     - 進入您的倉庫設置頁面
     - 選擇`SSH and GPG keys`
     - 點擊`New SSH key`
     - 將公鑰內容粘貼到`Key`字段中
     - 為密鑰添加`Title`，例如`id_rsa_vitepress`
     - 點擊`Add SSH key`保存密鑰  

## 管理多個賬號GitHub倉庫的SSH密鑰
   - 若您有其他 GitHub 賬號，需要為每個賬號創建一個獨立的密鑰對
   - 每個密鑰對的文件名稱應包含賬號名稱，例如`id_rsa_vitepress`、`id_rsa_butterfly`等
   - 編輯`~/.ssh/config`文件（若不存在，則創建一個），添加以下內容
```bash
Host github-vitepress
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_vitepress

Host github-butterfly
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_butterfly
```

![GitHub登录凭据.png](https://cloudflare-imgbed-7oz.pages.dev/file/1758920815738_GitHub登录凭据.png)

## 刪除 HTTPS 登錄憑證，改用 SSH 登錄（博客鑰匙）
   - 刪除 HTTPS 登錄憑證后，git clone `文檔站原石` 到本地
   - 打開終端或命令提示符
   - 執行以下命令克隆`文檔站原石`到本地
     ```bash
     git clone git@github-vitepress:<您的用戶名>/vitepress.git
     ```
   - 確認登錄憑證已替換
     ```bash
     git remote -v
     ```
     應該會看到類似以下的輸出：
     ```bash
     origin  git@github-vitepress:<您的用戶名>/vitepress.git (fetch)
     origin  git@github-vitepress:<您的用戶名>/vitepress.git (push)
     ```
   - 若不是以上的輸出：
     - 打開終端或命令提示符
     - 執行以下命令將遠程倉庫地址替換為 SSH 地址
     ```bash
     git remote set-url origin git@github-vitepress:<您的用戶名>/vitepress.git 
     ```
## 將原工作流全數刪除，寫入GitHub Actions全自動部署工作流
   - 刪除倉庫`.github/workflows/`文件夾裏的所有文件
   - 在倉庫`.github/workflows/`文件夾中創建一個名為`deploy.yml`的文件
   - 填入以下內容
```yaml
# 构建 VitePress 站点并将其部署到 GitHub Pages 的示例工作流程
#
name: Deploy VitePress site to Pages

on:
  # 在针对 `main` 分支的推送上运行。如果你
  # 使用 `master` 分支作为默认分支，请将其更改为 `master`
  push:
    branches: [main]

  # 允许你从 Actions 选项卡手动运行此工作流程
  workflow_dispatch:

# 设置 GITHUB_TOKEN 的权限，以允许部署到 GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# 只允许同时进行一次部署，跳过正在运行和最新队列之间的运行队列
# 但是，不要取消正在进行的运行，因为我们希望允许这些生产部署完成
concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  # 构建工作
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0 # 如果未启用 lastUpdated，则不需要
      - uses: pnpm/action-setup@v3 # 如果使用 pnpm，请取消此区域注释
        with:
          version: 9
      # - uses: oven-sh/setup-bun@v1 # 如果使用 Bun，请取消注释
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: pnpm # 或 npm / yarn
      - name: Setup Pages
        uses: actions/configure-pages@v4
      - name: Install dependencies
        run: pnpm install --no-frozen-lockfile # 或 npm ci / yarn install / bun install
      - name: Build with VitePress
        run: pnpm run docs:build # 或 pnpm docs:build / yarn docs:build / bun run docs:build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: docs/.vitepress/dist

  # 部署工作
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    needs: build
    runs-on: ubuntu-latest
    name: Deploy
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

![GitHub全自動部署-17.png](https://cloudflare-imgbed-7oz.pages.dev/file/1758993334601_GitHub全自動部署-17.png)

## 修改文檔站base
  - 打開 `docs/.vitepress/config.ts` 文件
  - 尋找 `export default defineConfig`
  - 在函數裏填入 `base: '/vitepress/',`

## Commit and Push
  ```bash
  git add .
  git commit -m "github action update"
  git push origin main
  ```