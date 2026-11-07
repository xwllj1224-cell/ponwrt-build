# PonWrt 云编译工程（XG-040G-MD / XG-040G-MF）

用 GitHub Actions 云编译带 **XG-PON 光口支持** 的 PonWrt 固件（基于 ImmortalWrt，源码 `zhoufla/ponwrt`，上游 `pbs05/ponwrt`）。

本机没有 Linux 环境（无 WSL / Docker），全部编译在 GitHub 托管 runner（ubuntu-24.04，4 核 16G）上完成。

## 文件

| 文件 | 作用 |
| --- | --- |
| `.github/workflows/build-ponwrt-md.yml` | **XG-040G-MD（AN7581，光口）** 构建流水线，手动触发，可选源码仓库/分支 |
| `.github/workflows/build-ponwrt.yml` | XG-040G-MF（AN7583）构建流水线 |
| `.github-extra.config` | 追加到 `configs/an7583.config` 之后的附加包配置（MF 用参考） |

## MD 流水线要点

- 源码默认 `zhoufla/ponwrt@master`：比上游多 10 个提交（apk 源指向南大镜像、PON 光模块温度、PPE 卸载状态、CPU 温度等）。
- 设备裁剪：`configs/an7581.config` 原本一次编 12 个机型的 multi-profile，脚本注释掉烽火/创维/兆能等 10 个，
  只保留 `nokia_xg-040g-md-ubi`、`nokia_xg-040g-md-ubi-usb-sfp`、`nokia_xg-040g-tf-ubi`。
  每个设备各编一份内核，裁掉 10 个约省 40~60 分钟。
- 失败快检：`make defconfig` 后立刻校验设备数与关键包，异常直接退出，不浪费 3 小时。
- 产物瘦身：只上传 `bin/targets/airoha/an7581/` 下 `*xg-040g-md*` 的镜像 + `sha256sums` + `profiles.json`（约几十 MB），
  不含整包 ipk 目录（上次全量约 1.65 GB）。

## 附加软件包

LuCI + 简体中文 + Argon 主题；Passwall、OpenClash、Tailscale（含 community 版）；
USB 上网卡/5G 模块全套（cdc-ncm、rndis、cdc-ether、cdc-mbim、qmi-wwan、usb-storage(+uas)、
usb-serial(+option)、usb-modeswitch(+data)、uqmi、umbim、comgt）；
https-dns-proxy、openssh-sftp-server、curl、htop、nano。

## 使用

1. Actions 页选 **Build PonWrt for XG-040G-MD** → Run workflow（源码仓库/分支可留默认）；
2. 冷编译约 3.5 小时（MD 实测 3h34m；MF 曾跑满 6 小时上限被取消，注意 `timeout-minutes`）；
3. 产物在该 run 的 Artifacts（`ponwrt-md-<编号>`）；
4. 刷机用 **`nokia_xg-040g-md-ubi` 的 `sysupgrade.itb`**；若改过硬件加了 USB/SFP 口则用 `-usb-sfp` 变体。

## 刷机步骤（同 MF 实测流程）

1. telnet 备份全分区（`bosa`、`ri` 是这台机器的命根子，必须用本机自己的）；
2. 用 [AN758x-Stock2UBI](https://github.com/pbs05/an758x-stock2ubi) 刷 U-Boot → Web 恢复界面 192.168.0.1；
3. 顺序：ubi-rebuild → flash-uboot → flash-board-data(bosa/ri) → flash(sysupgrade.itb) → boot-production；
4. LuCI → 网络 → PON → Configuration → PON board data 恢复 `bosa` / `ri`（第 3 步已传则核对）；
5. 填认证（移动：SN + PLOAM；电信：LOID + 密码；联通：LOID），PON 状态 O5 常驻即成功。

## 已知坑

- 工作流里 `sed` 用 `s|...|` 时，正则中的 `(a|b|c)` 会被当作分隔符，报 `unknown option to 's'` —— 改用 `s@...@`。
- MF（AN7583）+ 全套包 + 调制解调器包会跑满 6 小时上限，需要裁设备或减包。
- 上游 `release.yml` 用到 `environment: release` 与 `PONWRT_APK_PRIVATE_KEY` 签名密钥，fork 里没有，直接跑会缺签名。
