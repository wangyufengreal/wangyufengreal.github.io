---
title: 如何使用 GitHub Pages 创建个人博客
date: 2025-07-10 11:25:00 +0800
categories: [技术]
tags: [博客]
---

## Why Blog?

- `Just for Fun`.

- `Love and Share` 的重要渠道.

- 定期输出内容, 有助于形成正反馈, 发展学习与探索的积极性.

- 系统性整理知识, 便于快速温习, 应对即时的挑战, 人的记忆毕竟是不可靠的.

## Why GitHub Pages?

- **免费**: 免去了购买服务器和域名所带来的成本.

- **易用**: 只需要维护一个公开的 [GitHub](https://github.com/) 仓库,  往指定文件夹里写 `Markdown` 格式的文章即可, 每 `push` 一次 `commit`, 仓库会使用 [GitHub Actions](https://github.com/features/actions) 自动构建博客页面, 没有任何多余步骤.

## Step 1: 找到你最爱的主题🥰

- 在 [Jekyll Themes](http://jekyllthemes.org/) 中, 包含数百个风格迥异的主题, 也许我们在担心找不到自己最爱的主题以前, 更应该先考虑下自己是否有选择困难症😅.

- 本博客采用的主题是 [jekyll-theme-chirpy](http://jekyllthemes.org/themes/jekyll-theme-chirpy/), 点击 `Homepage` 按钮即可进入该主题的 `demo` [仓库](https://github.com/cotes2020/jekyll-theme-chirpy/), 接下来在 `README.md` 中找到 `Documentation` 部分, 进入文档, 就能够找到该主题的 [starter](https://github.com/cotes2020/chirpy-starter) 仓库, 这是我们博客的起点.

## Step 2: 创建你博客的仓库🧱

- 首先确保自己的 `GitHub` 仓库中没有名为 `你的用户名.github.io` 的仓库, 如果有, 请先 `Delete this repository`.

- 在主题的 [starter](https://github.com/cotes2020/chirpy-starter) 仓库中, 点击右上角的 `Use this template` 按钮, 然后选择 `Create a new repository`, 在 `Repository name` 中, 输入 `你的用户名.github.io` , 然后 `Create repository` 即可.

## Step 3: 在你的电脑上创建写博客的地方📰

- 首先进入你想写博客的路径, 执行以下命令:

```bash
git clone https://github.com/你的用户名/你的用户名.github.io.git
```

- 然后进入这个仓库即可, 本人使用 [VsCode](https://code.visualstudio.com/) 管理仓库, 使用其它 `IDE` 或者直接用 `记事本` 编辑也并没有什么区别.

## Step 4: 博客的初始配置⚙️

- 在根目录下的 `_config.yml` 中修改以下关键信息

    * `title`: 你的博客标题.
    * `tagline`: 一句简洁的描述或标语.
    * `description`: 更详细的网站描述，有助于 `SEO`.
    * `url`: 非常重要，修改为 `https://你的用户名.github.io`.
    * `author`: 你的名字或昵称.

- 其它如头像, `logo` 等, 请按需配置, 官方文档已经说的非常详细.

## Step 5: 上传第一篇文章✍🏻

- 文章应写在 `_posts` 目录下, 命名规范为 `YYYY-MM-DD-你的文章标题.md`, 如 `2025-07-10-my-first-post.md`.

- 在文章的开头, 应添加 `---` 包围的 `YAML` 配置块, 里面是文章的元数据.

```yaml
---
title: 我的第一篇博文
date: 2025-07-10 10:00:00 +0800
categories: [技术, 随笔]
tags: [博客, Jekyll]
---
```

- 然后在下面用常规的 `Markdown` 语法写上正文即可.

- 文章完成以后, 执行以下操作, 将内容推送至 `GitHub` 仓库.

```bash
git add .
git commit -m "feat: 初始化博客并发布第一篇文章"
git push
```

- 推送成功以后, `GitHub Actions` 会开始自动构建静态博客网站, 这一过程大约会消耗 1~2 分钟, 在仓库的 `Actions` 标签页中可以查看构建进度.

- 访问 `https://你的用户名.github.io`, 就可以看到你的第一篇文章啦🥰!

## 一些最佳实践

- 本地预览

```bash
bundle install
bundle exec jekyll serve --port 8099
```

- 使用自定义域名

    - 首先准备好域名, 然后在 `Cloudflare` 设置4条 `A` 记录, 将根节点 `@` 指向到 `GitHub Pages` 的 `ip`, 分别为 `185.199.108.153/185.199.109.153/185.199.110.153/185.199.111.153`. 
    - 然后在 `Cloudflare` 设置1条 `CNAME` 记录, 将 `www` 指向 `你的用户名.github.io` 即可.
    - 正常等待5分钟左右即可生效, 注意为了 `GitHub` 的 `ssl` 证书能够成功申请, 记录一开始要设置为 `DNS only` 模式.

- 在 `GitHub` 仓库中, 我们在 `Settings` 中找到 `Pages` 设置项, 将其中的 `Custom domain` 设置为带 `www` 的你的域名, 设置完以后, 要等到显示 `DNS check successful`, 且勾选 `Enforce HTTPS` 复选框, 说明设置成功, 此时域名已经可以访问.

- 最后再去打开 `Cloudflare` 记录的 `Proxied` 模式, 就可以安全地使用自己的域名啦.

## The End

* `Love and Share`.
