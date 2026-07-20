# cma-project

This template should help get you started developing with Vue 3 in Vite.

## Recommended IDE Setup

[VS Code](https://code.visualstudio.com/) + [Vue (Official)](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## Recommended Browser Setup

- Chromium-based browsers (Chrome, Edge, Brave, etc.):
  - [Vue.js devtools](https://chromewebstore.google.com/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd)
  - [Turn on Custom Object Formatter in Chrome DevTools](http://bit.ly/object-formatters)
- Firefox:
  - [Vue.js devtools](https://addons.mozilla.org/en-US/firefox/addon/vue-js-devtools/)
  - [Turn on Custom Object Formatter in Firefox DevTools](https://fxdx.dev/firefox-devtools-custom-object-formatters/)

## Type Support for `.vue` Imports in TS

TypeScript cannot handle type information for `.vue` imports by default, so we replace the `tsc` CLI with `vue-tsc` for type checking. In editors, we need [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) to make the TypeScript language service aware of `.vue` types.

## Customize configuration

See [Vite Configuration Reference](https://vite.dev/config/).

## Project Setup

```sh
pnpm install
```

### Compile and Hot-Reload for Development

```sh
pnpm dev
```

### Type-Check, Compile and Minify for Production

```sh
pnpm build
```

### Lint with [ESLint](https://eslint.org/)

```sh
pnpm lint
```

### dist deploy 
方法一：
# 在cma项目外城新建部署目录，绑定gh-pages分支
1. git worktree add ../gh-pages-deploy gh-pages
# 打包
2. npm run build
# 将部署目录 ../gh-pages-deploy清空
3. Remove-Item ../gh-pages-deploy/* -Recurse -Force
# 复制dist全部内容到gh-pages-deploy根目录中
4. Copy-Item dist/* ../gh-pages-deploy/ -Recurse
# 进入部署目录提交推送
5. cd ../gh-pages-deploy
6. git add .
7. git commit -m '更新静态打包文件'
8. git push origin gh-pages --force
9. cd ../cma-project

方法二：
# 安装gh-pages插件
1. npm install -g gh-pages  
# 将dist目录内容推送到远程仓库，其远程仓库的分支名一定要为gh-pages，下面代码的第一个gh-pages为远程分支名，第二个gh-pages为自定义分支名
2. npx gh-pages -d dist -b gh-pages -r https://github.com/wzxGithub-98/cma-project.git
