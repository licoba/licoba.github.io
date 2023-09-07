# HexoSource

> GitHub 的 Hexo 博客源码

## 安装 Hexo

参考中文官网：https://hexo.io/zh-cn/

在终端执行命令

```
npm install hexo-cli -g
hexo init blog
cd blog
npm install
hexo server
```

## 在 CloudStudio 里预览

参考 CloudStudio 的文档：https://cloudstudio.net/docs/guide/preview.html

按下 `Command + Shift + P` 或 `Ctrl + Shift + P`，打开命令面板，输入 `preview`，在命令列表中点击 `Preview: Generate Preview Config File`，就可以预览了。

## 主题

- 所有主题：https://hexo.io/themes/

#### 自动主题

- fluid: https://github.com/fluid-dev/hexo-theme-fluid （推荐）
- nexmoe: https://github.com/theme-nexmoe/hexo-theme-nexmoe
- aomori: https://github.com/lh1me/hexo-theme-aomori
- stellar:https://github.com/xaoxuu/hexo-theme-stellar （推荐）

#### 手动主题

- yun: https://github.com/YunYouJun/hexo-theme-yun
- obsidian: https://github.com/bennyxguo/hexo-theme-obsidian 
- snippet: https://github.com/shenliyang/hexo-theme-snippet
- cards: https://github.com/ChrAlpha/hexo-theme-cards.git

### 手动添加主题

```
cd blog
git submodule add https://github.com/YunYouJun/hexo-theme-yun themes/yun
git submodule add https://github.com/shenliyang/hexo-theme-snippet themes/snippet
git submodule add https://github.com/ChrAlpha/hexo-theme-cards.git themes/cards
git submodule add https://github.com/TriDiamond/hexo-theme-obsidian.git themes/obsidian
```

## 部署

执行命令

```
cd hexo
hexo g -d
```

## 站点

- GitHub: https://licoba.github.io
