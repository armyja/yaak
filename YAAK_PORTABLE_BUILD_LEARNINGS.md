# Yaak 便携版构建经验教训

> 记录所有失败原因，避免重复犯错

## 构建失败记录

| 日期 | 错误类型 | 错误信息 | 解决方案 | 教训 |
|------|----------|----------|----------|------|
| 02-03 | 配置错误 | `signCommand: ""` 导致 `program path has no file name` | 移除空签名配置 | 不要添加官方没有的字段 |
| 02-03 | 路径错误 | `Cannot find path 'src-tauri/target/release/'` | 使用 `crates-tauri/yaak-app/` | 要理解项目结构 |
| 02-03 | 脚本缺失 | `Missing script: "app-build-portable"` | 添加脚本到 package.json | 先检查脚本是否存在 |
| 02-04 | 配置字段 | `Additional properties are not allowed ('webview2' was unexpected)` | 不添加未知字段 | 只用官方配置中有的字段 |
| 02-04 | Node版本 | `Vite requires Node.js version 20.19+` | workflow 用 `node-version: '20'` | 先检查依赖版本要求 |
| 02-04 | CLI安装 | `cargo install tauri-cli` 编译30+分钟 | 改用 `npm install @tauri-apps/cli` | 选择更快的方案 |
| 02-04 | 参数传递 | `unexpected argument '--portable' found` | 用 `--` 传递参数 | tauri build -- --flag |
| 02-04 | 签名问题 | Azure签名失败 | 使用 `--no-sign` | CI环境跳过签名 |
| 02-04 | 配置文件路径 | `The system cannot find the file specified` | 使用完整路径 `crates-tauri/yaak-app/tauri.portable.conf.json` | 注意工作目录 |

## 正确的配置方式

### 方法1：使用 tauri-apps/tauri-action（推荐）
```yaml
- uses: tauri-apps/tauri-action@v0
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    args: "--bundles nsis"
```

### 方法2：直接构建（参考 Docsee GUI）
```yaml
- name: Build Tauri app
  run: |
    npm run tauri build -- --bundles nsis
```

## 关键教训

1. **先调研，再动手**
   - 查看官方 release.yml 配置
   - 参考成熟项目（Docsee GUI）
   - 不要盲目尝试

2. **理解后再修改**
   - 了解项目结构（Yaak 用 crates-tauri/yaak-app/）
   - 理解 tauri 命令的参数传递（用 `--`）
   - 知道 Tauri v1/v2 的配置差异

3. **简单最好**
   - 不要添加复杂的配置
   - 能用默认就用默认
   - NSIS 安装包本身就可以作为"准绿色版"

4. **测试验证**
   - 先在本地测试
   - 不要直接推送到 CI
   - 每次只改一个东西

## 便携版方案总结

### 最简单的方案
1. 构建 NSIS 安装包
2. 用户用 7-Zip 解压安装包 = 便携版

### 为什么这个方案好
- ✅ 不需要复杂的便携配置
- ✅ NSIS 包本身是自包含的
- ✅ 解压后直接运行，无需安装
- ✅ 官方支持，稳定可靠

## 参考项目

- Docsee GUI: https://github.com/Xczer/docsee-gui
  - 使用 `bun tauri build --bundles msi,nsis`
  - workflow 配置简单可靠
