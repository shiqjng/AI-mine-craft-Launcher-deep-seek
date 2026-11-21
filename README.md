# AI-mine-craft-Launcher-deep-seek
# 鲸屿 · Whale Isle

一个小而全的 Minecraft 桌面启动器。Electron + 原生前端，不需要 Java 环境就能跑；装游戏、装加载器、找模组、拖整合包，一个 exe 全包了。

- 从官方源 / BMCLAPI 镜像安装游戏本体，自动测速选源
- Fabric / Quilt / Forge / NeoForge 四种加载器，装游戏时一起装好
- Modrinth 与 CurseForge 搜索下载模组，自动处理必需/可选前置
- Modrinth / Planet Minecraft 找资源包，Modrinth / ShadersMods 找光影
- 投影工坊（mcimg.com）找投影，装进 `schematics`
- 拖入 jar / zip / 文件夹 / 整合包直接装
- 微软正版账户登录，或离线账户
- 版本目录是 HMCL 那种「一个游戏版本一个文件夹」

当前版本：**1.1.2**

---

## 功能

### 游戏本体

- 官方源（`piston-meta.mojang.com` / `resources.download.minecraft.net`）与 BMCLAPI 镜像双通道。
- **自动测源**：每次安装前各下载一次资源索引，谁快用谁；结果缓存 30 分钟，安装过程里也会显示「下载源测速：官方 x.xs / 镜像 x.xs」。
- 完整安装：版本 json、客户端 jar、`libraries`、资源索引、3800+ 个资源对象、natives 解压。
- 进度条显示百分比、已下载 / 总量、实时速度、剩余时间，支持取消。
- 下载带 SHA1 与大小校验、失败自动换源重试；大文件走 Node 原生下载，避免 Chromium 拿到响应头后卡死。
- 默认并发 32，可在设置里改。

### 加载器

- 安装面板可选 **原版 / Fabric / Quilt / Forge / NeoForge**，选好版本点安装即可。
- Fabric、Quilt 直接写 profile 并下载组件；Forge、NeoForge 调官方安装器，安装前会从镜像预下载安装器依赖（NeoForge 的 Maven 在国内经常不可达，做了多镜像回退）。
- 装完自动合并成一个文件夹：`versions/26.3-Fabric/`，里面同时有 `26.3-Fabric.json`、`26.3-Fabric.jar`、`natives`、`mods`、`saves`。

### 模组

- Modrinth / CurseForge 两个来源，按当前实例的游戏版本与加载器过滤。
- 安装前弹「安装前确认前置」：必需前置勾死并强制安装，缺少必需前置时不允许继续；可选前置单独标注，允许不装。
- 已经装过的前置不再提示、不会重复下载（靠 projectId 和 jar 内 `fabric.mod.json` 的 id 双判，手动丢进去的也能认出来）。
- 重复安装检测：同一个 jar 或同一项目已有更高版本时会提示「已经安装过 / 已有更高版本」，可以选择跳过或仍然安装。
- 启动器启动后自动检查已装模组的更新，有更新在右下角和首页提示，可一键更新。
- 支持启用 / 禁用 / 删除与本地 jar 导入。

### 资源包 / 光影 / 投影

- 资源包：Modrinth + Planet Minecraft。PMC 有 Cloudflare 人机校验，抓不到时提供「浏览器打开 + 粘贴 zip 直链导入」的降级方案。
- 光影：Modrinth + ShadersMods，ShadersMods 项目页解析出 CurseForge 直链直接下载，不需要 API Key。光影需要 Iris 或 OptiFine。
- 投影：从投影工坊（mcimg.com）搜索下载，装进实例的 `schematics` 目录；支持子文件夹结构与中文文件名（GBK / GB18030 兜底）。
- 四个内容页右上角都有「目标版本」下拉，多版本时直接切换要操作的实例。

### 拖入导入

把文件或文件夹拖到窗口里即可：

| 拖入内容 | 行为 |
| --- | --- |
| `.jar` | 装进当前实例的 `mods`，先做重复检测 |
| 资源包 `.zip` | 装进 `resourcepacks` |
| 光影 `.zip` | 装进 `shaderpacks` |
| 含投影的文件夹 / 压缩包 | 递归找投影，装进 `schematics` |
| 整合包 `.zip` / `.mrpack` | 识别格式，安装本体与加载器后解压配置、下载模组 |

整合包支持 CurseForge `manifest.json`、Modrinth `modrinth.index.json`、MultiMC 与普通「放好 mods 的压缩包」。CurseForge 格式的模组需要 API Key 才能批量下载，没填会提示并只导入整合包自带文件。

### 账户

- 离线账户：输入名字即可，UUID 按 `OfflinePlayer:<name>` 生成。
- 微软正版：内置登录窗口，走 OAuth 授权码 → Xbox Live → XSTS → Minecraft Services，令牌过期自动用 refresh_token 续期，可加入正版服务器、同步皮肤。
- 密码只在微软页面输入，启动器不接触、不保存；只存 `accessToken` / `refreshToken`（本机 `config.json`，不会上传）。

### Java

- 自动扫描注册表（JavaSoft / Adoptium / Microsoft / Zulu / BellSoft / Corretto）、`Program Files`、`ProgramData\chocolatey`、`~/.jdks`、scoop 以及 `.minecraft\runtime`。
- 版本不匹配时可一键下载 Java 8 / 11 / 17 / 21 / 25：默认 GitHub，失败自动切华为云、清华镜像。
- 启动前按版本 json 的 `javaVersion.majorVersion` 选择，取「满足要求的最低版本」。

### 启动

- 自动拼 JVM / 游戏参数、classpath、natives、日志配置，处理 26.x 的 `java.library.path` 子目录与临时目录。
- 从别处拷进来的版本缺库时，启动前自动补齐 `libraries` 与 `natives`（见 1.0.19）。
- 实时日志、崩溃原因分类（Java 版本不符 / 缺核心 / 内存不足 / 模组冲突等）。
- 原版核心目录里有模组时会提示「原版加载不了这些模组」，防止误启动。

---

## 快速开始

1. 把 `鲸屿.exe` 放到一个**普通可写目录**（不要在 `Program Files` 里）。
2. 双击运行，首次会解压约 30 秒，期间没有窗口，属正常。
3. 打开「版本」页，选一个游戏版本；需要模组就切到 Fabric / Quilt / Forge / NeoForge，点安装。
4. 去「模组 / 资源包 / 光影 / 投影」页搜索安装，右上角可切换目标实例。
5. 回「首页」点「启动游戏」。

> 想更快：设置里把「下载源」保持「自动」，并发下载数 32；访问 Modrinth 慢时填本地代理。

---

## 数据目录

默认全部放在 exe 旁边：

| 内容 | 位置 |
| --- | --- |
| 配置、账户、窗口状态 | `<启动器目录>\config.json` |
| 浏览器缓存（登录态） | `<启动器目录>\.session` |
| 游戏文件 | `<启动器目录>\.minecraft`（可在设置里改） |
| 版本 | `.minecraft\versions\<版本>` |
| 启动日志 | `.minecraft\logs\launch-<时间>.log` |
| 资源缓存 | `.minecraft\assets`、`.minecraft\libraries`（所有版本共享） |

装在不可写位置时会自动回退到 `%APPDATA%\WhaleIsle`；从旧版本升级会把旧配置搬过来，已有的游戏目录不动。

关闭「一个实例一个文件夹」后，内容直接放在 `.minecraft\{mods,resourcepacks,shaderpacks}`，和原版启动器一致。

### 版本目录长这样

```
.minecraft/
  assets/            # 所有版本共享
  libraries/         # 所有版本共享
  versions/
    26.3-Fabric/
      26.3-Fabric.json
      26.3-Fabric.jar
      natives/
      mods/
      resourcepacks/
      shaderpacks/
      schematics/
      saves/
      logs/
      config/
```

一个游戏版本只有一个文件夹。手动装模组时直接进 `versions/26.3-Fabric/mods` 就行。

---

## 正版账户

「设置 → 账户 → 添加微软账户（正版）」会打开微软官方登录页（Minecraft 品牌页）：

1. 登录后启动器依次完成：授权码换 MSA token → Xbox Live 认证 → XSTS 授权 → Minecraft Services 登录 → 读取档案。
2. 登录令牌过期时启动前自动刷新；刷新失败会提示重新登录。
3. 账户必须已购买 Minecraft（Java 版），没有档案会提示「该微软账户没有购买 Minecraft」。
4. 正版账户启动使用 `--accessToken` / `--uuid` / `--userType msa` / `--xuid`，可加入正版服务器并同步皮肤；离线账户仍是 `--userType legacy`，只能单机 / 局域网。
5. 在账户卡片上点「使用」可切回离线账户；删除正版账户会同时清掉本机令牌。

如果登录报错，把 `<启动器目录>\auth-debug.log` 发过来，里面按步骤记了每一步的响应。

---

## 下载与源

| 设置 | 说明 |
| --- | --- |
| 自动（测速选源） | 安装前各下一次索引，选快的那个；推荐 |
| 仅 BMCLAPI 镜像 | 强制走镜像 |
| 仅官方源 | 强制走 Mojang 官方 |

- 资源文件多是 10–20KB 的小文件，并发 32 时本地实测能到 8–10 MB/s；整个 1.20.4（626MB 资源 + 24MB 客户端 + 依赖库）在干净目录里约 1–2 分钟。
- 单个文件失败会自动换下一个地址/通道（Chromium fetch ↔ Node 原生流），并清理 0 字节残留。
- 超时策略：20 秒无数据判失败，180 秒无进展放弃并重试。

---

## 第三方站点

- **Modrinth**：公开 API，无需配置。
- **CurseForge**：官方 API 需要 Key，在「设置 → CurseForge」填写（[申请入口](https://forms.monday.com/forms/dce5ccb7afda9a1c21dab1a1aa1d84eb?r=use1)，需本人提交，审核后邮件发 Key）。没 Key 也能用 Modrinth。
- **Planet Minecraft**：Cloudflare 人机校验，程序抓不到时请用「打开网页 + 粘贴 zip 直链」导入。
- **ShadersMods**：可直接抓搜索与项目页，下载走 CurseForge 直链。
- **投影工坊（mcimg.com）**：站点 `/shimu/api` 公开接口，附件匿名为可下载。
- **NeoForge Maven**：国内常有 DNS 污染 / 不可达，安装器依赖会先从 BMCL、EngineHub、Stellardrift 等镜像预下载。

---

## 常见问题

**从 HMCL 把版本文件夹拷过来，启动报 `ClassNotFoundException: net.fabricmc.loader.impl.launch.knot.KnotClient`？**
只拷了 `versions\xxx`，`libraries` 里的 fabric-loader 没跟过来。1.0.19 起启动前会自动补齐缺失的库和 natives，直接点启动即可。

**拷过来的版本被认成「原版」，还提示原版加载不了模组？**
HMCL 的合并版 json 没有 `inheritsFrom`，也没有本启动器的 `.mc-launcher.json`。1.0.18 起会看 `mainClass` 和 `libraries` 判断 Fabric / Quilt / Forge / NeoForge。

**exe 双击没反应？**
先确认不在受限目录（如本项目的开发目录 `F:\MC启动器`）。便携版首次运行要解压约 30 秒，期间没有窗口；也可以试试 `dist\win-unpacked\鲸屿.exe`。

**Windows 提示「已保护你的电脑」？**
exe 没有代码签名，点「更多信息 → 仍要运行」。

**装在 Program Files 里配置丢了？**
那里不可写，启动器会回退到 `%APPDATA%\WhaleIsle`，把 exe 挪到普通目录即可。

**游戏启动后崩溃？**
看「首页 → 启动日志」或 `.minecraft\logs\launch-*.log`，崩溃分类会给出大致方向；模组冲突优先查重复安装与前置缺失。

---

## 开发

```bash
npm install
npm start            # 开发运行
npm run dist         # 打包便携版 exe（输出到 dist/）
npm run dist:installer  # 打包 NSIS 安装包
```

需要 Node 18+。打包用 electron-builder 的 portable 目标，产物是单文件 exe。

开发时若报 `app.getPath` 相关错误，先清掉环境变量 `ELECTRON_RUN_AS_NODE`，否则 Electron 会以纯 Node 模式运行。

### 目录结构

```
electron/
  main.js            # 主进程、IPC、窗口、单实例
  preload.js         # contextBridge 暴露 window.mc
  core/
    net.js           # Chromium net.fetch + Node 原生流双通道下载
    mirrors.js       # 官方 / BMCLAPI / Maven 地址映射与源顺序
    store.js         # 配置持久化、实例目录、数据迁移
    tasks.js         # 下载/安装任务与进度
    versions.js      # 版本清单、加载器 profile、已安装列表
    installer.js     # 本体安装、libraries、assets、natives、加载器安装器
    layout.js        # HMCL 式单文件夹布局与迁移
    launch.js        # 参数拼装、进程与日志
    java.js          # Java 检测与 Adoptium 下载
    auth.js          # 微软正版登录与令牌续期
    modrinth.js      # Modrinth 搜索/版本/依赖
    curseforge.js    # CurseForge 搜索/文件/依赖
    content.js       # 实例内容扫描、导入、重复检测
    modpacks.js      # 整合包识别与导入
    schematics.js    # 投影工坊接口
    updates.js       # 模组更新检查
renderer/
  index.html  styles.css
  js/{app,state,dom,actions,dropzone,deps,updates-ui,instance-select}.js
  js/views/{home,versions,mods,packs,shaders,schematics,settings}.js
```

### 开发调试

- `.tools/cdp-eval.js`：通过 CDP 在渲染层执行 JS，`--shot` 截图，`--drag` 模拟文件拖入，`--targets` 列出页面。
- `.tools/bump-version.js`：改版本号（纯 Node，避免 PowerShell 写坏中文）。
- `.tools/make-icon-photo.js`：从图片生成圆角图标与多尺寸 ico。

> 注意：不要用 PowerShell 5.1 的 `Get-Content -Raw | Set-Content -Encoding UTF8` 改含中文的文件，会按 ANSI 读坏。

---

- **启动前兼容检查**：启动前扫一遍模组元数据（含内置 jar-in-jar），发现「加载器版本太低 / 模组要别的游戏版本」会先弹窗，可一键禁用冲突模组、升级 Fabric Loader 后再启动，不用等进游戏才报错。

- **加载器升级**：版本页每个加载器实例都有扳手按钮，可以单独把 Fabric / Quilt / Forge / NeoForge 升到新版本，只换加载器本体，模组、存档、配置原地不动。
- **模组更新**：检查更新时只在「适配当前基础游戏版本 + 加载器」的版本里挑版本号最高的，不会把旧版本补发的小版本误当成更新。

## 联机

鲸屿的联机走 SakuraFrp 隧道，房间码只负责把「地址 + 版本」传给朋友，不经过我们的服务器。

1. 房主在游戏里按 Esc →「对局域网开放」，记下端口。
2. 在 SakuraFrp 里给这个端口建一条 TCP 隧道并开启，复制它给的连接地址（如 `frp-abc.com:10086`）。
3. 打开鲸屿的「联机 → 创建房间」，启动器会自动检测到局域网端口，粘贴地址后点「生成房间码」——房间码形如 `frp-abc.com:10086#26.3-Fabric`，生成后会直接复制到剪贴板。
4. 把房间码发给朋友；朋友在「加入房间」粘贴（或点「从剪贴板粘贴」，启动器也会自动读取剪贴板）并点「解析并加入」，启动器会用本地的对应版本带上 `--quickPlayMultiplayer` 直接进服。

注意：房间码里只有地址和版本信息，双方版本 / 模组要一致，进不去先检查版本。房主退出世界，房间也就断了。
## 更新日志

### 1.1.2（正式发布）

- 导入整合包后，模组检测改用「Minecraft 本体版本」：以前整合包名（例如 Closing Song1.6.7.1）被当成基础版本，模组页、更新检查、前置解析全按这个名字过滤，结果一个都匹配不上。
- 修正方式：从版本 json 里读真实本体版本与加载器版本（inheritsFrom、--fml.mcVersion、net.minecraftforge:forge:1.20.1-47.3.22 这几处），老实例下次启动会自动改写 .mc-launcher.json，游戏目录与存档不动。
- 识别整合包里自带的版本文件夹（PCL2 导出的「版本文件夹」或整个 .minecraft），导入时按本体版本装原版 + 加载器，不再把整合包名当版本号。
- 版本列表里整合包实例显示整合包名，副标题标出「本体 1.20.1」，路径仍是 versions/<整合包名>。
- 修整合包启动崩溃 `Invalid paths argument, contained no existing paths`：自包含版本 json 引用的 client-srg / client-extra / forge-client.jar 只能由官方安装器生成，启动前检测到缺失会自动跑一遍 Forge / NeoForge 安装器补齐，补完把临时建出来的版本目录清掉，不污染版本列表。
- 自包含版本缺资源索引时自动补 assets（1.20.1 首次约 609 MB、3526 个文件），日志里能看到百分比，不再进游戏一片没贴图。
- 下载器允许不传 AbortSignal（内部补资源用），不会再因为缺 signal 直接抛错。
- 启动前兼容检查按实例的加载器读元数据：Forge / NeoForge 读 `META-INF/mods.toml`（Maven 区间），Fabric / Quilt 才读 `fabric.mod.json`；不再拿 Fabric 的声明去判断 Forge 整合包，弹窗文案也跟着变。
- 修版本区间误判：`26.3.x`、`[1.20.1]`、`[1.18,1.20.4]`、`[47,)`、逗号列表、`1.18 || 1.20.1` 以及没做变量替换的 `${...}` 占位符都能正确处理，认不出来的一律当满足，不再吓人。
### 1.1.1（正式发布）

- 模组更新进度条改成按字节走：百分比、已下载 / 总量、实时速度、剩余时间都在动，不再一直停在 0%。
- 一次更新多个模组时按总大小累计进度，任务栏能看到「2/4 · 模组名」。
- 每个更新条目右侧加了「更新日志」，弹窗显示新版本改了什么（Modrinth 是 Markdown，CurseForge 走文件日志接口）；更新完成后也有结果弹窗，可以逐个点开看。
- 更新可以随时取消；取消不再误报「更新失败」，下载中的半截文件会一起清掉。
- 单个文件 90 秒没有任何字节进展就换下一个下载通道重试；上次异常退出留下的 .part-* 在下次下载前自动清理。
- 修内容索引被清空的问题：以前读目录失败会把整份索引当成残留记录删掉，导致模组来源与项目 id 丢失、更新检查只能退化成哈希反查。
- 正式版第一次启动会弹窗列出这个版本改了什么（内测包不弹），内容由 README 的更新日志生成。

### 1.1.0（正式发布）

- 新增「联机」：SakuraFrp 隧道 + 房间码，创建房间/加入房间、自动检测局域网端口、粘贴房间码直接进服。
- 房间码改成短格式（`frp-abc.com:10086#26.3-Fabric`），生成立即复制到剪贴板；加入房间会自动读取剪贴板，也支持一键粘贴。
- 启动前模组兼容检查：Fabric / Quilt 加载器版本太低、模组要求更高的游戏版本时，直接列出问题，可以「修复并启动」（禁用冲突模组 + 升级加载器）。
- 版本页新增「升级加载器」按钮，Fabric / Quilt / Forge / NeoForge 可单独升级，模组、存档、配置原地不动。
- 模组更新只在「适配当前基础版本 + 加载器」的版本里挑版本号最高的，不再把旧版本补发当成更新。
- 启动游戏后隐藏改为收到系统托盘，点托盘图标随时能打开（联机房主需要回来填地址），游戏退出后自动弹回；从联机页进服不会隐藏。
- 搜索请求超时会自动重试一次，不再误报「没有找到」。

### 1.0.26
- 模组、资源包、光影、投影四个列表改成翻页，不再一路往下滚（投影工坊每页 10 条，其它每页 20 条，已安装每页 15 条）。
- 深色主题下的下拉框（加载器版本、目标版本）改成深底亮字，原先白底白字看不清。
- 内测包版本号带 `-beta.N`，界面左下角显示 BETA 标签。

### 1.0.25（维护）
- 模组、资源包、光影、原理图的列表改为分页浏览（页码 + 上一页/下一页），不再用「加载更多」往下叠。
- 已安装列表同样分页，每页 15 条。
- 维护改动记录见 docs/维护履历.md。

### 1.0.24
- 「启动游戏后隐藏启动器」：勾选后启动游戏会把窗口收到托盘，点托盘图标随时能再打开（联机房主开完局域网还要回来填 SakuraFrp 地址），游戏退出后自动弹回。

### 1.0.23
- 账户头像改由启动器自己下载皮肤裁头部并缓存，不再依赖第三方头像站；加载前显示名字首字母。

### 1.0.22
- 设置 → 账户里给正版账户加了「皮肤」按钮：选 PNG、选经典/纤细手臂、上传，也可以恢复默认皮肤。

### 1.0.21
- 启动参数增加 -Djava.net.preferIPv4Stack=true，缓解部分网络下 Java 连 Mojang 服务超时导致的皮肤不显示。

### 1.0.20
- 下载提速：安装前自动测速选源；`libraries` 也优先官方源；默认并发 16 → 32；资源并发不低于 32。
- 1.20.4 干净目录实测从约 13 分钟降到 1–2 分钟。

### 1.0.19
- 启动前自动补齐缺失的库与 natives，修 HMCL 拷贝版本报 `ClassNotFoundException ... KnotClient`。

### 1.0.18
- 修 HMCL 拷贝版本被认成「原版」：看 `mainClass` / `libraries` 判断加载器。

### 1.0.17
- 迁移残留的旧版本目录（被游戏占用没删干净）会自动合并后再删。

### 1.0.16
- 版本列表里的目录提示显示真实路径（`versions/26.3-Fabric`）。

### 1.0.15
- 版本目录改成 HMCL 风格：一个游戏版本一个文件夹，本体与加载器合并，不再「本体 + 加载器」两份。

### 1.0.14
- 同一游戏版本只显示一条，装了加载器后不再单列原版。

### 1.0.13
- 模组 / 资源包 / 光影 / 投影页右上角新增「目标版本」下拉。

### 1.0.12
- 清空目录后不再显示幽灵条目（索引会剔除不存在的文件）。

### 1.0.11
- 投影支持子文件夹递归扫描；已安装数量与刷新同步。

### 1.0.10
- 投影目录随文件变化刷新；GBK / GB18030 文件名不再乱码；下载名去掉时间戳前缀。

### 1.0.9
- 支持拖入文件夹与压缩包导入投影；目录与压缩包自动识别。

### 1.0.8
- 新增「原理图」页：从投影工坊搜索、下载、安装到 `schematics`。

### 1.0.7
- 拖入导入重复检测：已存在 / 已有更高版本会提示。

### 1.0.6
- 修整合包导入点「开始导入」没反应、进度条显示错误单位。

### 1.0.5
- 启动时检查 Minecraft 新版本（含快照），右下角提示。

### 1.0.4
- 修拖入时拿不到文件路径（FileList 跨 contextBridge 丢路径）。

### 1.0.3
- 新增拖入导入与整合包导入（CurseForge / Modrinth / MultiMC）。

### 1.0.2
- 图标换成鲸鱼娘头像。

### 1.0.1
- 一个实例一个文件夹，模组 / 材质 / 光影 / 存档统一在版本目录里。

### 1.0.0
- 更名「鲸屿」，正式版。

> 1.0.0 之前的 1.2.x–1.5.x 均为测试版。

