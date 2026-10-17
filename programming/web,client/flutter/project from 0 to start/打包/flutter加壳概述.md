# 🛡️ Flutter `libapp.so` 内存加载加壳全流程终极指南

本方案通过“**物理剔除 + 内存还原 + 运行时劫持**”的技术闭环，确保业务逻辑不以明文形式出现在磁盘上。其核心在于利用 **JNI 桥接** 抢在 Flutter 引擎启动前，通过内存 FD 完成阵地部署与符号拦截。

---

## 一、 静态加密阶段 (编译期：数据隐身)
**任务：** 将 AOT 产物从“可执行文件”降级为“加密数据包”。

1.  **AOT 编译**：执行 `flutter build apk --release`，获取原始 `libapp.so`。
2.  **认证加密 (AES-256-GCM)**：
    *   使用 Python 脚本生成随机 **12B IV**（初始化向量）。
    *   加密产出密文并生成 **16B Tag**（完整性校验码）。
    *   **安全性**：GCM 模式确保了数据即使被篡改，在解密时也会被立即发现。
3.  **资源隐写**：将封装好的二进制流（IV + Tag + Ciphertext）存入 `assets/shells/`，彻底改变其文件特征。

---

## 二、 壳代码开发阶段 (运行期：执行心脏)
**任务：** 编写 C++ 层的 `libloader.so`，实现从内存还原代码的完整链路。

1.  **主动防御 (ptrace)**：调用 `ptrace(PTRACE_TRACEME)`。若应用已被附加调试，此操作将失败并触发自杀逻辑。
2.  **JNI 跨界桥接**：
    *   在 Java 层定义 `native` 接口，接收加密字节流。
    *   通过 `System.loadLibrary("loader")` 建立 JNI 映射，将执行权由 JVM 移交给 C++ 环境。
3.  **内存加载双技**：
    *   **memfd_create**：创建匿名内存文件。该文件不属于任何磁盘路径，完美绕过 Android 10+ 严苛的 SELinux 路径策略。
    *   **android_dlopen_ext**：直接通过文件描述符（FD）加载库，实现代码在 RAM 中直接映射。
4.  **隐身重定向 (ShadowHook)**：拦截 `libc.so` 的 `dlopen`。当 Flutter 引擎寻找 `libapp.so` 时，壳程序直接投喂预先加载好的内存句柄。

---

## 三、 物理篡改与注入阶段 (重构期：启动劫持)
**任务：** 利用 **Apktool** 对 APK 进行“外科手术”，强制更改应用启动生命周期。

1.  **外科拆解**：使用 `apktool d` 反编译，将二进制 XML 与 DEX 转化为可读的文本与 Smali 逻辑。
2.  **物理清除**：**强行删除** `lib/[ABI]/libapp.so`。这是确保静态扫描无法发现源码的物理基础。
3.  **劫持入口 (Manifest)**：修改 `AndroidManifest.xml`，将 `application` 的启动入口改为壳类 `ShieldApplication`。
4.  **Smali 注入**：
    *   植入 `NativeLoader.smali`（JNI 桥接类）。
    *   植入 `ShieldApplication.smali`，通过继承原 Application 并重写 `attachBaseContext`，确保壳逻辑在所有组件前运行。

---

## 四、 自动化流水线阶段 (封装期：闭环生成)
**任务：** 使用 Shell 脚本实现一键化的加壳与重组。

1.  **连贯动作块**：在脚本内实现原件删除、壳 SO 植入、Smali 代码注入与 XML 修改的同步完成。
2.  **回编译与对齐**：
    *   **apktool b**：重组 APK 结构。
    *   **zipalign**：强制执行 4 字节对齐，这是 Android 系统内存加载 SO 的刚性要求。
3.  **重签名 (apksigner)**：由于物理篡改破坏了原签名，必须进行 V2/V3 重新签名，以保证安装权限与系统完整性校验。

---

## 🚀 核心价值与攻防博弈总结


| 维度 | 实现手段 | 防护等级 | 核心价值 |
| :--- | :--- | :--- | :--- |
| **静态审计** | 物理剔除 | **极高** | 逆向者在 APK 目录内找不到任何业务 SO。 |
| **路径监控** | memfd 机制 | **极佳** | 避开所有基于路径（如 /data/local/tmp）的检测规则。 |
| **动态注入** | ptrace + Hook | **强** | 阻止调试器挂载，实现对闭源引擎的隐身投喂。 |
| **完整性** | AES-GCM | **高** | 任何对加密 bin 文件的修改都会导致解密失败并中止运行。 |

---

### 💡 最终建议：
该方案已实现工业级的代码隐身。为进一步提升破解难度，建议对 `libloader.so` 进行 **OLLVM (Obfuscator-LLVM)** 指令平坦化混淆，隐藏 AES 密钥与 Hook 逻辑的跳转路径。
