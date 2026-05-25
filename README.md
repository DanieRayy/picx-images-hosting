# picx-images-hosting

该项目用于存储图片，作为免费的线上图片仓库使用。

这个仓库也是 `juya-news-card` 的图片图床仓库。  
作用是把本地 `public/uploads/...` 的手动粘贴图片，转成可公网访问的 CDN 链接，主要用于 `issuePost.md` 发布到 GitHub Issue 时正常显示图片。

## 核心用途

- 存放从 `juya-news-card` 自动同步过来的图片素材。
- 通过 jsDelivr 提供外链，例如：
  `https://cdn.jsdelivr.net/gh/DanieRayy/picx-images-hosting@main/imagehub/curate2/...`
- 避免在 GitHub Issue / 远程环境里出现本地路径（如 `/uploads/...`）不可见的问题。

## 什么时候会用到

- 在 `curate2` 页面里手动粘贴上传了图片。
- 执行了 `npm run pipeline:export -- --date YYYYMMDD`。
- 需要把 `pipeline/final/YYYYMMDD/issuePost.md` 发到 GitHub Issue，并确保图片可见。

不需要它的场景：
- 仅本地预览、仅本地渲染视频，不对外发布 Issue。
- 所有图片都已经是公网 URL（不含 `/uploads/...` 本地路径）。

## 工作方式（自动）

`juya-news-card/scripts/pipeline-export.ts` 在导出时会：

1. 检查是否有 `/uploads/...` 本地图片。
2. 自动 clone/更新本仓库到本地工作目录（默认 `juya-news-card/.cache/picx-images-hosting`）。
3. 复制图片到本仓库目录（默认前缀：`imagehub/curate2`）。
4. `git add/commit/push` 到 `main` 分支。
5. 把 `issuePost.md` 内链接改写为 jsDelivr CDN 链接。

## 目录约定

默认路径形态：

```text
imagehub/curate2/{date}/{id}/{filename}
```

例如：

```text
imagehub/curate2/20260506/3/1778079060911-1cf0c40b0f1a.png
```

对应 CDN：

```text
https://cdn.jsdelivr.net/gh/DanieRayy/picx-images-hosting@main/imagehub/curate2/20260506/3/1778079060911-1cf0c40b0f1a.png
```

## 配置（在 `juya-news-card/.env`）

```env
# 是否启用自动同步（默认 true）
PICX_SYNC_ENABLED=true

# 图床仓库信息
PICX_OWNER=DanieRayy
PICX_REPO=picx-images-hosting
PICX_BRANCH=main

# 可选：自定义仓库地址与本地工作目录
# PICX_REPO_URL=https://github.com/DanieRayy/picx-images-hosting.git
# PICX_WORKDIR=.cache/picx-images-hosting

# 图床仓库中的存放前缀
# PICX_BASE_DIR=imagehub/curate2
```

## 使用步骤

1. 在 `juya-news-card` 完成 `curate2` 编辑并保存（含手动图片）。
2. 运行：
   `npm run pipeline:export -- --date YYYYMMDD`
3. 看到日志：
   `已同步图片素材到图床并改写 issuePost 链接（张数：N）`
4. 检查 `pipeline/final/YYYYMMDD/issuePost.md` 是否已是 `cdn.jsdelivr.net` 链接。

## 常见问题

- 没有改写成 CDN：
  通常是没有本地 `/uploads/...` 图片，或 `PICX_SYNC_ENABLED=false`。
- 同步失败但导出未中断：
  脚本会降级保留原 `/uploads/...` 链接，并打印告警（常见原因是 git push 权限不足）。
- CDN 刚同步后暂时 404：
  jsDelivr 可能有短暂缓存延迟，稍等片刻再访问。
