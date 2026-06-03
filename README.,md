# 超星学习通（chaoxing.com）视频 & PPT 下载方法

## 核心原理

超星学习通的视频和 PPT 附件全部由 JS 动态加载，URL 不直接出现在静态 HTML 中。下载的关键路径是：

```
课程页面 → iframe[info="card"] → mArg.attachments → ananas/status API → cldisk CDN 直链
```

## 步骤总览

### 第一步：在浏览器控制台获取附件信息

进入课程的视频/PPT 播放页面，F12 → Console，粘贴以下脚本并运行：

```javascript
(function() {
    var cardIframe = document.querySelector('iframe[info="card"]');
    if (!cardIframe) { console.log('请进入课程视频/PPT页面'); return; }
    var win = cardIframe.contentWindow;
    if (!win || !win.mArg) { console.log('无法访问数据'); return; }

    var mArg = win.mArg;
    var atts = mArg.attachments || [];
    var defs = mArg.defaults || {};

    if (atts.length === 0) { console.log('本章节无附件'); return; }

    atts.forEach(function(att, i) {
        var p = att.property || {};
        var objId = att.objectId || p.objectid;
        var isVideo = att.type === 'video';

        console.log('\n' + (isVideo ? '🎬' : '📄') + ' ' + p.name + ' (' + (p.hsize || '') + ')');
        console.log('  objectId:', objId);
        console.log('  aid:', att.aid);
        console.log('  mid:', att.mid);

        fetch('https://mooc1.chaoxing.com/ananas/status/' + objId
            + '?k=' + defs.fid + '&flag=normal&_dc=' + Date.now(),
            { credentials: 'include' })
            .then(function(r) { return r.json(); })
            .then(function(data) {
                console.log('  ananas/status 完整响应:', JSON.stringify(data, null, 2));
            });
    });
})();
```

输出中重点关注以下字段：

| 字段 | 用途 |
|------|------|
| `data.download` | CDN 下载接口（`d0.cldisk.com/download/...`） |
| `data.http` | CDN 流式直链（`s2.cldisk.com/.../sd.mp4`，只能播不能下） |
| `data.pdf` | PPT 转 PDF 直链（文档类型） |
| `data.dtoken` | 下载 token（需配合 Cookie 使用） |
| `data.filename` | 原始文件名 |

### 第二步：下载 PPT

PPT 的 `ananas/status` API 返回中有 `pdf` 字段，是无需 Cookie 的 CDN 直链，直接用 curl 下载：

```powershell
curl.exe -L -o "文件名.pdf" "https://s3.cldisk.com/sv-w3/doc/.../pdf/xxx.pdf"
```

或者用 `download` 字段下载原始 PPT：

```powershell
curl.exe -L -o "文件名.ppt" -H "Referer: https://mooc1.chaoxing.com/" "https://d0.cldisk.com/download/xxx?at_=...&ak_=...&ad_=..."
```

### 第三步：下载视频

视频的 `ananas/status` API 返回中有 `download` 字段，指向 `d0.cldisk.com/download/...`，这是真正的下载接口，**无需 Cookie**，只需加 Referer 头：

```powershell
curl.exe -L -o "01.mp4" -H "Referer: https://mooc1.chaoxing.com/" "https://d0.cldisk.com/download/9e757273f1c418864eb23b7753c6fe77?at_=1780206629798&ak_=5d54270e324adee855d6a02c2d5a4aef&ad_=d3adfbeed2362257906a2af046371bb2"
```

**注意**：`at_` / `ak_` / `ad_` 这三个 token 有时效性，获取后尽快下载。

## 各类 URL 的可用性对比

| URL 类型 | 示例 | 用途 | 需要 Cookie？ |
|----------|------|------|:---:|
| `d0.cldisk.com/download/...?at_=&ak_=&ad_=` | CDN 下载接口 | **下载视频/PPT** ✅ | ❌ 不需要 |
| `s3.cldisk.com/.../pdf/xxx.pdf` | CDN PDF 直链 | **下载 PPT PDF** ✅ | ❌ 不需要 |
| `s2.cldisk.com/.../sd.mp4?at_=&ak_=&ad_=` | CDN 流式播放 | 浏览器能播但不能下载 ⚠️ | ❌ 不需要 |
| `d0.ananas.chaoxing.com/download/...` | 超星下载接口 | 403 被拒 ❌ | ✅ 需要 |
| `d0.ananas.chaoxing.com/{id}/{dtoken}/...` | dtoken 拼接 | 403 被拒 ❌ | ✅ 需要 |

## 完整脚本

完整的浏览器端提取 + 下载脚本见 `chaoxing_extract.js`。

## 其他方案尝试记录

| 方案 | 结果 |
|------|------|
| yt-dlp + `--cookies-from-browser edge/chrome` | ❌ Windows DPAPI 加密错误，无法提取 Cookie |
| Python + `browser_cookie3` 库 | ❌ 需要管理员权限 |
| Python 直接请求超星 API | ❌ 无登录 Cookie，被重定向到登录页 |
| 浏览器 CORS XHR 下载 | ❌ `d0.ananas` 跨域 CORS 阻止 + 403 |
| **curl + cldisk download 接口** | ✅ **成功！886 MB 视频 2 分 48 秒下载完成** |

## 关键发现

1. 超星的真实存储后端是 **CLdisk**（云盘），不是 ananas
2. `ananas/status` 是获取下载 URL 的核心 API，同源无 CORS 问题
3. `d0.cldisk.com/download` 是最终的下载接口，带 token 参数无需 Cookie
4. `d0.ananas.chaoxing.com` 的任何接口都需要浏览器 Cookie，命令行无法使用
5. 视频的 `data.http` 是流式播放链接（`Content-Disposition: inline`），浏览器打开会播放而非下载；必须用 `data.download` 才是下载专用链接
