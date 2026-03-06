# ClawList 图床使用指南

## 设置完成

✅ **GitHub 图床仓库**: https://github.com/Tony11081/clawlist-images  
✅ **本地路径**: `~/clawd/projects/clawlist-images/`  
✅ **上传脚本**: `upload-clawlist-image` (已安装到 PATH)

## 使用方法

### 方法 1: 使用上传脚本（推荐）

```bash
# 上传单张图片（自动命名）
upload-clawlist-image image.png

# 上传到指定路径
upload-clawlist-image image.png blog/my-article/cover.png

# 批量上传
for img in *.png; do
  upload-clawlist-image "$img" "blog/my-article/$img"
done
```

**输出示例：**
```
✅ Image uploaded!

📎 URL: https://raw.githubusercontent.com/Tony11081/clawlist-images/main/blog/image.png

Markdown:
![Image](https://raw.githubusercontent.com/Tony11081/clawlist-images/main/blog/image.png)

HTML:
<img src="https://raw.githubusercontent.com/Tony11081/clawlist-images/main/blog/image.png" alt="Image" />
```

### 方法 2: 手动上传

```bash
# 1. 复制图片到仓库
cp image.png ~/clawd/projects/clawlist-images/blog/

# 2. 提交并推送
cd ~/clawd/projects/clawlist-images
git add blog/image.png
git commit -m "Add image: blog/image.png"
git push origin main

# 3. 使用 URL
# https://raw.githubusercontent.com/Tony11081/clawlist-images/main/blog/image.png
```

## 配图工作流（新流程）

### 1. 生成图片

```bash
# 使用 baoyu-danger-gemini-web 生成
SKILL_DIR=~/clawd/skills/baoyu-danger-gemini-web
npx -y bun ${SKILL_DIR}/scripts/main.ts \
  --prompt "Your prompt" \
  --image output.png
```

### 2. 上传到 GitHub 图床

```bash
# 上传单张
upload-clawlist-image output.png blog/article-slug/01-image.png

# 批量上传（配图场景）
cd illustrations/article-slug/
for img in *.png; do
  upload-clawlist-image "$img" "blog/article-slug/$img"
done
```

### 3. 更新文章内容

```markdown
![Image description](https://raw.githubusercontent.com/Tony11081/clawlist-images/main/blog/article-slug/01-image.png)
```

### 4. 更新数据库

```bash
# 直接更新 Supabase
node -e "
const { createClient } = require('@supabase/supabase-js');
const fs = require('fs');

const supabase = createClient(
  'https://ygnbikloljpjzkxxcoar.supabase.co',
  'YOUR_SERVICE_ROLE_KEY'
);

const content = fs.readFileSync('article.md', 'utf8');

supabase
  .from('blog_posts')
  .update({ content })
  .eq('slug', 'article-slug')
  .then(({ error }) => {
    if (error) throw error;
    console.log('✅ Updated!');
  });
"
```

## 优势

**对比之前的流程：**

| 步骤 | 旧流程（Vercel public/） | 新流程（GitHub 图床） |
|------|-------------------------|---------------------|
| 上传图片 | 复制到 public/ | 上传到 GitHub 仓库 |
| 提交代码 | Git commit + push | Git commit + push |
| 部署等待 | ⏳ Vercel 重新构建（1-2分钟） | ✅ 即时可用（无需构建） |
| 图片 URL | /blog/image.png | raw.githubusercontent.com/... |
| 更新文章 | 需要重新部署 | 只更新数据库 |

**新流程优势：**
- ✅ 图片即传即用，无需等待部署
- ✅ 不触发 Vercel 重新构建
- ✅ 图片和代码分离，仓库更轻量
- ✅ GitHub CDN 全球加速
- ✅ 可以独立管理图片版本

## 目录结构

```
clawlist-images/
├── blog/
│   ├── anthropic-plugins/
│   │   ├── 01-infographic-saas-extinction.png
│   │   ├── 02-infographic-plugin-tiers.png
│   │   ├── 03-infographic-deployment-strategy.png
│   │   └── 04-infographic-iteration-speed.png
│   ├── article-2/
│   │   └── cover.png
│   └── article-3/
│       ├── 01-intro.png
│       └── 02-conclusion.png
├── skills/
│   └── skill-screenshots/
└── misc/
    └── logos/
```

**命名规范：**
- 文章配图：`blog/{article-slug}/NN-{type}-{description}.png`
- 封面图：`blog/{article-slug}/cover.png`
- Skill 截图：`skills/{skill-name}/screenshot.png`
- 其他资源：`misc/{category}/{name}.png`

## 完整示例：发布带配图的文章

```bash
# 1. 生成配图（4张）
cd ~/clawd
SKILL_DIR=~/clawd/skills/baoyu-danger-gemini-web

for i in {1..4}; do
  npx -y bun ${SKILL_DIR}/scripts/main.ts \
    --promptfiles prompts/0${i}.md \
    --image illustrations/my-article/0${i}-image.png
done

# 2. 批量上传到 GitHub 图床
cd illustrations/my-article/
for img in *.png; do
  upload-clawlist-image "$img" "blog/my-article/$img"
done

# 3. 编写文章（使用 GitHub 图床 URL）
cat > article.md << 'EOF'
# My Article

![Image 1](https://raw.githubusercontent.com/Tony11081/clawlist-images/main/blog/my-article/01-image.png)

Content...

![Image 2](https://raw.githubusercontent.com/Tony11081/clawlist-images/main/blog/my-article/02-image.png)
EOF

# 4. 发布到 ClawList
cd ~/clawd/projects/clawlist-app
NEXT_PUBLIC_SUPABASE_URL=https://ygnbikloljpjzkxxcoar.supabase.co \
SUPABASE_SERVICE_ROLE_KEY=xxx \
node -e "
const { createClient } = require('@supabase/supabase-js');
const fs = require('fs');

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL,
  process.env.SUPABASE_SERVICE_ROLE_KEY
);

const content = fs.readFileSync('article.md', 'utf8');

supabase
  .from('blog_posts')
  .insert({
    title: 'My Article',
    slug: 'my-article',
    summary: 'Article summary',
    content: content,
    category: 'AI',
    tags: ['tag1', 'tag2'],
    author: 'Author Name',
    published_at: new Date().toISOString(),
    reading_time: 5
  })
  .then(({ error }) => {
    if (error) throw error;
    console.log('✅ Published!');
    console.log('🔗 https://clawlist.io/blog/my-article');
  });
"
```

## 故障排除

**"Command not found: upload-clawlist-image"**
```bash
export PATH="$HOME/.local/bin:$PATH"
source ~/.zshrc
```

**"Image repo not found"**
```bash
cd ~/clawd/projects/clawlist-images
git remote -v
# 应该显示: origin  https://github.com/Tony11081/clawlist-images.git
```

**"Push rejected"**
```bash
cd ~/clawd/projects/clawlist-images
git pull origin main
git push origin main
```

## 记录到 MEMORY.md

已添加到工作流程：
- 图片生成：baoyu-danger-gemini-web
- 图片上传：upload-clawlist-image（GitHub 图床）
- 文章发布：直接更新 Supabase，无需重新部署

---

**状态**: ✅ 已配置完成  
**仓库**: https://github.com/Tony11081/clawlist-images  
**上次更新**: 2026-03-06
