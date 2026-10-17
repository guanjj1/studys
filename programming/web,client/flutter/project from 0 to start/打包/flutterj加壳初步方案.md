<!--
 * @Author: guanjiajun www.guanjiajun@ewake.com
 * @Date: 2026-04-17 17:39:50
 * @LastEditors: guanjiajun www.guanjiajun@ewake.com
 * @LastEditTime: 2026-04-18 11:23:01
 * @FilePath: \studys\programming\web,client\flutter\project from 0 to start\打包\flutterj加壳初步方案.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
# 第一阶段：静态加密 (编译期：数据隐身)

**任务**：将 AOT 产物从“可执行文件”降级为“加密数据包”，混淆文件特征并存储于 Assets 目录。

### Python 加密脚本细节 (`encrypt.py`)
```python
import os
from Crypto.Cipher import AES

# 32位密钥，必须与 C++ 解密层保持严格一致
KEY = b'your_32_byte_secure_key_12345678' 

def encrypt_so(abi_path, output_dir):
    so_file = os.path.join(abi_path, "libapp.so")
    if not os.path.exists(so_file):
        print(f"❌ 未找到: {so_file}")
        return

    # GCM 模式推荐使用 12 字节随机 IV
    iv = os.urandom(12) 
    cipher = AES.new(KEY, AES.MODE_GCM, iv)
    
    with open(so_file, 'rb') as f:
        plaintext = f.read()
        
    ciphertext, tag = cipher.encrypt_and_digest(plaintext)

    # 封装格式：IV(12B) + Tag(16B) + Ciphertext
    os.makedirs(output_dir, exist_ok=True)
    abi_name = os.path.basename(abi_path)
    output_path = os.path.join(output_dir, f"app_{abi_name}.bin")
    
    with open(output_path, "wb") as f:
        f.write(iv + tag + ciphertext)
    print(f"✅ 加密完成: {abi_name} -> {output_path}")
```

# 第二阶段：壳代码开发 (加载期实现)

**任务**：编写原生加载器 `libloader.so`，实现：反调试、依赖链提权、内存解密、无路径加载 (`memfd`) 以及符号劫持 (`ShadowHook`)。

### C++ 核心代码细节 (`loader.cpp`)
```cpp
#include <sys/mman.h>
#include <android/dlext.h>
#include <sys/ptrace.h>
#include <unistd.h>
#include <string.h>
#include "shadowhook.h"
#include "mbedtls/gcm.h"

static void* g_app_handle = nullptr;
static const unsigned char GLOBAL_KEY = {0x...}; // 与Python Key字节一致

// Hook 回调：当 Flutter 引擎尝试 dlopen("libapp.so") 时，返回内存中的句柄
static void* my_dlopen(const char* filename, int flags) {
    if (filename && strstr(filename, "libapp.so")) {
        return g_app_handle;
    }
    return shadowhook_dlopen(filename, flags);
}

extern "C" JNIEXPORT jboolean JNICALL
Java_com_example_app_NativeLoader_initShield(JNIEnv* env, jclass clazz, jbyteArray data) {
    // 1. 基础反调试抢占：防止调试器挂载
    if (ptrace(PTRACE_TRACEME, 0, 0, 0) < 0) return JNI_FALSE;

    // 1. 【核心补丁】符号提权：确保 libflutter.so 符号全局可见，防止 libapp.so 链接断裂
    // 使用 RTLD_NOLOAD 标志：若已加载则提升权限为 GLOBAL，若未加载则强制加载
    void* engine_h = dlopen("libflutter.so", RTLD_GLOBAL | RTLD_NOW);
    if (!engine_h) return JNI_FALSE;

    // 2. 内存解密：解析数据流 [IV(12) | Tag(16) | Ciphertext(...)]
    jsize len = env->GetArrayLength(data);
    jbyte* buf = env->GetByteArrayElements(data, nullptr);
    
    // 示例：使用 MbedTLS 执行 AES-GCM 解密并校验完整性
    // mbedtls_gcm_auth_decrypt(..., decrypted_ptr);

    // 3. 创建匿名 FD 加载（不落地、无磁盘路径）
    int fd = memfd_create("flutter_mem", MFD_CLOEXEC);
    write(fd, decrypted_ptr, plain_len);

    // 4. 利用 Android 特有 API 通过 FD 加载内存中的 SO
    android_dlextinfo extinfo;
    extinfo.flags = ANDROID_DLEXT_USE_LIBRARY_FD;
    extinfo.library_fd = fd;
    g_app_handle = android_dlopen_ext("libapp.so", RTLD_NOW, &extinfo);

    // 5. 开启 Hook 拦截，接管系统 dlopen 符号
    shadowhook_init(SHADOWHOOK_MODE_UNIQUE, false);
    shadowhook_hook_sym_name("libc.so", "dlopen", (void*)my_dlopen, nullptr);

    env->ReleaseByteArrayElements(data, buf, JNI_ABORT);
    return g_app_handle != nullptr;
}
```
# 第三阶段：物理篡改与注入 (重构期)

**任务**：物理剔除 APK 内的明文 SO，并利用 Smali 代码劫持 `Application` 启动入口。

### smali前的java代码(优先)
```java
package com.example.app;

import android.content.Context;
import android.os.Build;
import java.io.InputStream;

public class NativeLoader {
    static {
        // 1. 提权：确保引擎符号全局可见
        System.loadLibrary("flutter");
        // 2. 加载：加载我们的壳 SO
        System.loadLibrary("loader");
    }

    public static void init(Context context) {
        try {
            // 根据 ABI 读取密文资源
            String abi = Build.SUPPORTED_ABIS[0];
            InputStream is = context.getAssets().open("shells/app_" + abi + ".bin");
            byte[] data = new byte[is.available()];
            is.read(data);
            is.close();

            // 进入 C++ 环境执行解密与拦截
            if (!initShield(data)) {
                // 校验失败则自毁
                android.os.Process.killProcess(android.os.Process.myPid());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static native boolean initShield(byte[] data);
}

```
```java
package com.example.app; // 这里的包名要与你的脚本配置一致

import android.content.Context;
import io.flutter.app.FlutterApplication;

/**
 * 劫持入口类
 * 作用：在 Flutter 引擎启动前，强制执行解密和 Hook 逻辑
 */
public class ShieldApplication extends FlutterApplication {

    @Override
    protected void attachBaseContext(Context base) {
        // 1. 先执行父类的初始化（必须放在第一行，确保系统 Context 准备就绪）
        super.attachBaseContext(base);

        // 2. 立即启动壳逻辑
        // 这一步会触发 NativeLoader 类的加载，进而触发 static 块中的 System.loadLibrary
        // 然后执行 initShield(data)，在内存中布好 ShadowHook 的陷阱
        try {
            NativeLoader.init(base);
        } catch (Exception e) {
            // 如果加壳逻辑崩溃，建议直接退出，防止泄露未受保护的环境
            android.os.Process.killProcess(android.os.Process.myPid());
        }
    }
}

```
#### kotlin版本
```kotlin
package com.example.app

import android.content.Context
import android.os.Build

object NativeLoader {
    init {
        // 提权与加载
        System.loadLibrary("flutter")
        System.loadLibrary("loader")
    }

    @JvmStatic
    fun init(context: Context) {
        try {
            val abi = Build.SUPPORTED_ABIS[0]
            val encryptedData = context.assets.open("shells/app_$abi.bin").use { 
                it.readBytes() 
            }
            if (!initShield(encryptedData)) {
                android.os.Process.killProcess(android.os.Process.myPid())
            }
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }

    @JvmStatic
    private external fun initShield(data: ByteArray): Boolean
}

```
```kotlin
package com.example.app

import android.content.Context
import io.flutter.app.FlutterApplication

class ShieldApplication : FlutterApplication() {
    override fun attachBaseContext(base: Context) {
        super.attachBaseContext(base)
        // 启动壳
        NativeLoader.init(base)
    }
}

```

### Smali 注入细节 (`ShieldApplication.smali`)
```smali
.class public Lcom/example/app/ShieldApplication;
.super Lio/flutter/app/FlutterApplication; # 关键：必须继承原 APK 实际的 Application 类

.method protected attachBaseContext(Landroid/content/Context;)V
    .locals 0
    # 1. 调用父类逻辑 (确保系统 Context 初始化)
    invoke-super {p0, p1}, Lio/flutter/app/FlutterApplication;->attachBaseContext(Landroid/content/Context;)V
    
    # 2. 调用壳逻辑：由 NativeLoader 执行读取 Asset -> JNI 解密加载
    invoke-static {p1}, Lcom/example/app/NativeLoader;->init(Landroid/content/Context;)V
    return-void
.end method
```

# 第四阶段：自动化流水线 (封装期)

**任务**：通过 Shell 脚本将所有步骤串联，实现一键打包即加壳。

### 自动化脚本细节 (`shield_all_in_one.sh`)
```bash
#!/bin/bash

# 1. 正常编译产物
flutter build apk --release
# 编译 C++ loader 生成 libloader.so

# 2. 加密业务代码
python3 encrypt.py

# 3. 反编译劫持
apktool d build/app/outputs/flutter-apk/app-release.apk -o build/dist -f

# 4. 修改 Manifest 劫持入口类
sed -i 's/android:name="io.flutter.app.FlutterApplication"/android:name="com.example.app.ShieldApplication"/g' build/dist/AndroidManifest.xml

# 5. 植入 Smali 逻辑代码
mkdir -p build/dist/smali/com/example/app
cp scripts/smali_template/*.smali build/dist/smali/com/example/app/

# 6. 外科手术：物理置换 SO
find build/dist/lib -name "libapp.so" -delete
cp build/intermediates/libloader.so build/dist/lib/arm64-v8a/

# 7. 注入加密资源二进制文件
mkdir -p build/dist/assets/shells
cp build/outputs/app_arm64-v8a.bin build/dist/assets/shells/

# 8. 重新打包、对齐、签名
apktool b build/dist -o build/unsigned.apk
zipalign -v 4 build/unsigned.apk build/aligned.apk
apksigner sign --ks release.jks --ks-pass pass:123456 --out build/final.apk build/aligned.apk

echo "🎉 全流程加壳打包完成！"
```
# ✅ 壳程序实施最终 Checklist

### 1. 物理层检查 (静态审计)
- [ ] **SO 剔除确认**：使用 `unzip -l final.apk` 查看 `lib/` 目录，确保 `libapp.so` 彻底消失。
- [ ] **壳 SO 植入**：确认 `libloader.so` 出现在对应的 ABI 文件夹下。
- [ ] **密文资源**：确认 `assets/shells/` 目录下包含加密后的 `.bin` 文件。

### 2. 逻辑层检查 (运行时审计)
- [ ] **KEY 一致性**：确认 Python 脚本的加密 KEY 与 C++ 代码中的解密 KEY 字节序列 100% 匹配。
- [ ] **继承关系**：确认 `ShieldApplication.smali` 继承的是原 APK 清单文件里定义的旧 Application 类。
- [ ] **入口劫持**：确认 `AndroidManifest.xml` 的 `android:name` 正确指向了 `com.example.app.ShieldApplication`。

### 3. 系统兼容性检查
- [ ] **内存对齐**：必须在签名之前执行过 `zipalign`，否则 Android 10+ 无法加载内存中的 FD。
- [ ] **签名验证**：必须使用 `apksigner` 生成 V2/V3 签名。
- [ ] **多架构**：如果 App 支持多种 ABI，确保每个文件夹下的 SO 都已正确处理。

### 4. 运行验证
- [ ] **冷启动测试**：应用能否正常打开？
- [ ] **日志监控**：`adb logcat` 是否显示 `dlopen` 拦截成功？
- [ ] **反调试验证**：连接调试器时，应用是否按预期崩溃/退出？
