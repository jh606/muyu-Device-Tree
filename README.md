# muyu — Xiaomi TWRP Device Tree

基于小米 **HyperOS 4.0.3**（`OS4.0.3.0.XOYCNXM`）OTA 镜像提取生成的 TWRP 设备树。

## 设备信息

| 项 | 值 |
|---|---|
| 代号 | `muyu` |
| 平台 | Qualcomm **SM8635**（CliffsP，`qcom,msm-id = 0x282`，soc_id 642） |
| 内核 | `6.1.157-android14-11-g2de7246565ac-mi`（GKI，prebuilt） |
| boot header | v4，pagesize 4096，ramdisk 0 |
| 分区 | A/B + super 动态分区，独立 `recovery` 分区 |
| 文件系统 | system/vendor 等 **erofs**，userdata **f2fs**（FBE wrapped-key） |

## 生成方式

用 [twrpdtgen](https://github.com/twrpdtgen/twrpdtgen) 3.0.0 生成，输入全部来自本机 OTA 镜像：

| 输入 | 来源 |
|---|---|
| kernel | `boot.img` 中 36,751,872 B 的 ARM64 `Image` |
| dtb | `vendor_boot.img` 的 dtb 段（4 个 SoC 基础树中 msm-id=0x282 那个）+ `dtbo.img` 的 134 个 overlay 合并 |
| dtbo | `dtbo.img` 原样（594,812 B 有效数据 + AVB） |
| fstab | `vendor_boot.img` 里的 `first_stage_ramdisk/fstab.qcom` |
| cmdline | base dtb 的 `chosen/bootargs` |

> 注：twrpdtgen 的 CLI 依赖 AIK，而 AIK 的 `unpackbootimg` 不认 header v4（实测返回码 1），因此改用它的 `DeviceTree.dump_to_folder()` 生成器，模板与目录结构仍由 twrpdtgen 输出。

## 目录

```
.
├── Android.bp / Android.mk / AndroidProducts.mk
├── BoardConfig.mk          # 板级配置（v4 / pagesize / cmdline 均为实测值）
├── device.mk / omni_muyu.mk / vendorsetup.sh
├── recovery.fstab          # 由 stock fstab.qcom 转换（26 条）
├── extract-files.sh / setup-makefiles.sh
└── prebuilt/
    ├── kernel              # 6.1.157-android14-11-mi Image
    ├── dtb.img             # base3(CliffsP) + overlay 合并结果
    └── dtbo.img            # OTA 原始 dtbo
```

## 已知限制

1. `BOARD_SUPER_PARTITION_SIZE := 9126805504` 是 twrpdtgen 模板占位，需用 `blockdev --getsize64 /dev/block/by-name/super` 的真实值替换。
2. `extract-files.sh` 为模板骨架，vendor 闭源库需从 `vendor.img` / 真机抽取。
3. `ro.product.model`、安全补丁级别、fingerprint 为推断值（镜像中不含），可自行替换。
4. TWRP 若要解密 userdata，需补 `TW_INCLUDE_CRYPTO := true`、`TW_USE_FSCRYPT_POLICY := 2` 与 stock fstab 里的 `fileencryption=...wrappedkey_v0` 参数。

## 致谢

- [twrpdtgen](https://github.com/twrpdtgen/twrpdtgen) — 设备树生成器
- [TWRP](https://twrp.me/) — Team Win Recovery Project

## License

Apache-2.0

---

> 本仓库的设备树由 **DeepSeek Harness AI 助手**从设备镜像中提取、生成并整理，代为提交上传至 GitHub。

