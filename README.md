# arbeit-tools-public-release

这个仓库用于公开发布 Arbeit Tools 的安装包和插件包。

## 1. 两个关键地址

1. `PUBLIC_RELEASE_REPO`
   作用：应用更新发布镜像仓库（owner/repo）
   示例：`Jonas0124/arbeit-tools-public-release`
2. `AT_PLUGIN_MANIFEST_URL`
   作用：插件清单地址（raw 文件 URL）
   示例：`https://raw.githubusercontent.com/Jonas0124/arbeit-tools-public-release/main/plugins-manifest.json`

说明：

- 两者独立。
- `PUBLIC_RELEASE_REPO` 管应用版本发布。
- `AT_PLUGIN_MANIFEST_URL` 管插件下载信息（url/hash/size）。

## 2. 正常发版流程（应用）

1. 在私有应用仓库统一升级版本号：
   - `src-tauri/Cargo.toml`
   - `src-tauri/tauri.conf.json`
   - `package.json`（建议同步）
2. 提交并推送代码。
3. 打 tag 并推送：

```bash
git tag v1.1.0
git push origin v1.1.0
```

4. 等待 GitHub Actions：`Build & Release Desktop`。
5. `release_public` 成功后，本公开仓库会自动创建同名 release 并上传产物。

## 3. plugins-manifest.json 更新规则

### 3.1 插件文件未变化

只改顶层 `version` 也可以，不需要重新上传大文件。

### 3.2 插件文件有变化（或 URL/tag 变化）

1. 先把插件包上传到本仓库对应 release。
2. 对“最终发布的那个文件”计算 `sha256` 与 `size_bytes`。
3. 更新 `plugins-manifest.json` 对应字段：
   - `url`
   - `sha256`
   - `size_bytes`
   - `eta_seconds`
4. 提交 `plugins-manifest.json`。

## 4. sha256 和 size_bytes 计算命令

### Linux

```bash
FILE="poppler-windows-x64.zip"
sha256sum "$FILE"
wc -c < "$FILE"
```

### macOS

```bash
FILE="poppler-windows-x64.zip"
shasum -a 256 "$FILE"
stat -f%z "$FILE"
```

### Windows PowerShell

```powershell
$File = ".\\poppler-windows-x64.zip"
(Get-FileHash $File -Algorithm SHA256).Hash.ToLower()
(Get-Item $File).Length
```

## 5. 清单示例

```json
{
  "version": "v1.1.0",
  "plugins": {
    "poppler": {
      "windows-x64": {
        "url": "https://github.com/Jonas0124/arbeit-tools-public-release/releases/download/v1.1.0/poppler-windows-x64.zip",
        "sha256": "<64-hex>",
        "size_bytes": 15967510,
        "eta_seconds": 45
      }
    }
  }
}
```

## 6. 重要校验

1. `url` 指向的文件必须和你计算哈希的文件完全一致。
2. 文件内容只要变动，`sha256` 必须重算。
3. `AT_PLUGIN_MANIFEST_URL` 是应用构建时注入；改了地址后需要重新发布应用版本。
