# MF2 PonWrt 编译工程

用 GitHub Actions 云编译带 **XG-PON 光口支持** 的 XG-040G-MF 固件（ponwrt，pbs05/ponwrt，基于 ImmortalWrt）。

## 文件
- `.github/workflows/build-ponwrt.yml` —— 构建流水线（手动触发）
- `.github-extra.config` —— 追加到 `configs/an7583.config` 之后的附加包配置

## 固件内容
基础：ponwrt AN7583 配置（PON 驱动 + NPU 固件 + 双 profile：`nokia_xg-040g-mf` / `nokia_xg-040g-mf-ubi`）
附加：LuCI（中文）、Passwall、OpenClash、Tailscale、kmod-usb-net-cdc-ncm/rndis（MU300）、https-dns-proxy、sftp/curl/htop/nano

## 使用
1. 推送到 GitHub 仓库；
2. Actions 页选 "Build PonWrt for XG-040G-MF" → Run workflow；
3. 首次编译约 3~5 小时，产物在 run 的 Artifacts（`ponwrt-an7583-<编号>`）；
4. 刷机用 **`nokia_xg-040g-mf-ubi` 的 sysupgrade.itb**（走 stock2ubi → Web 恢复流程，同 MF1）。

## 刷机步骤（同 MF1 实测流程）
1. telnet 备份 MF2 全分区（bosa/ri 是这台机器的命根子，必须用 MF2 自己的）；
2. stock2ubi 刷 U-Boot → Web 恢复界面 192.168.0.1；
3. 顺序：ubi-rebuild → flash-uboot → flash-board-data(bosa/ri) → flash(sysupgrade.itb) → boot-production；
4. LuCI → 网络 → PON → Configuration → PON board data 恢复 bosa/ri（如果第 3 步已传则核对）；
5. 填认证（移动：SN + PLOAM；电信：LOID + 密码；联通：LOID），PON 状态 O5 常驻即成功。
