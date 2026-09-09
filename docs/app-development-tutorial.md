# MCphone Ultra 應用開發教學

本教學將教你如何開發一個 MCphone 手機應用，並上傳到資源站讓其他玩家下載。

## 目錄

1. [簡介](#簡介)
2. [環境準備](#環境準備)
3. [第一個 App：Hello World](#第一個-apphello-world)
4. [資料夾結構](#資料夾結構)
5. [SPI 清單](#spi-清單)
6. [語言檔](#語言檔)
7. [應用圖示](#應用圖示)
8. [進階：互動式頁面](#進階互動式頁面)
9. [編譯與打包](#編譯與打包)
10. [測試](#測試)
11. [上傳到資源站](#上傳到資源站)
12. [常見問題](#常見問題)

---

## 簡介

MCphone 應用是一個標準的 JAR 檔，包含：
- Java 程式碼（實作 `IPhoneApp` 介面）
- SPI 清單（告訴 MCphone 去哪裡找你的 App）
- 資源檔（語言檔、圖示等）

MCphone 會在啟動時自動掃描所有 JAR 檔，找到實作 `IPhoneApp` 的類別，並註冊到手機的應用列表中。

---

## 環境準備

你需要：
- **JDK 21**（Minecraft 1.21.1 需要 Java 21）
- **MCphone 1.9.1 JAR 檔**（當作編譯依賴）
- 文字編輯器或 IDE（IntelliJ IDEA、VS Code 等）

下載 MCphone JAR：
```
https://github.com/november521/mcphone/releases
```

---

## 第一個 App：Hello World

我們來做一個最簡單的 App，打開後顯示「Hello, MCphone!」。

### 步驟 1：建立資料夾

建立以下資料夾結構：

```
my-first-app/
├── META-INF/
│   └── services/
│       └── com.november.mcphone.api.client.app.IPhoneApp
├── com/
│   └── example/
│       └── HelloApp.java
└── assets/
    └── myfirstapp/
        ├── lang/
        │   ├── zh_cn.json
        │   └── en_us.json
        └── textures/
            └── app/
                └── hello.png
```

### 步驟 2：寫 App 程式碼

建立 `com/example/HelloApp.java`：

```java
package com.example;

import com.november.mcphone.api.client.app.IPhoneApp;
import com.november.mcphone.api.client.ui.IPhonePage;
import com.november.mcphone.api.client.ui.PhoneCanvas;
import net.minecraft.network.chat.Component;
import net.minecraft.resources.ResourceLocation;

public class HelloApp implements IPhoneApp {
    
    @Override
    public String getId() {
        return "myfirstapp:hello";
    }
    
    @Override
    public Component getDisplayName() {
        return Component.translatable("app.myfirstapp.hello");
    }
    
    @Override
    public ResourceLocation getIconTexture() {
        return ResourceLocation.fromNamespaceAndPath(
            "myfirstapp", 
            "textures/app/hello.png"
        );
    }
    
    @Override
    public void onPress(IPhoneApp.Context ctx) {
        ctx.openPage(new HelloPage());
    }
    
    public static class HelloPage implements IPhonePage {
        
        @Override
        public void render(PhoneCanvas canvas) {
            // 繪製背景
            canvas.fill(0, 0, canvas.width, canvas.height, 0xFF1a1a2e);
            
            // 繪製標題（置中）
            canvas.drawCentered(
                "Hello, MCphone!", 
                canvas.width / 2, 
                40, 
                0xFFFFFFFF
            );
            
            // 繪製一般文字
            canvas.text(
                "這是我的第一個 App！", 
                10, 
                80, 
                0xFFAAAAAA
            );
        }
    }
}
```

### 步驟 3：寫 SPI 清單

建立 `META-INF/services/com.november.mcphone.api.client.app.IPhoneApp`（**注意：沒有副檔名**）：

```
com.example.HelloApp
```

這個檔案告訴 MCphone：「我的 App 類別是 `com.example.HelloApp`，請註冊它」。

如果你的 JAR 裡有多個 App，每行寫一個類別名稱。

### 步驟 4：寫語言檔

建立 `assets/myfirstapp/lang/zh_cn.json`：

```json
{
  "app.myfirstapp.hello": "哈囉世界",
  "app.myfirstapp.hello.desc": "我的第一個 MCphone 應用"
}
```

建立 `assets/myfirstapp/lang/en_us.json`：

```json
{
  "app.myfirstapp.hello": "Hello World",
  "app.myfirstapp.hello.desc": "My first MCphone app"
}
```

### 步驟 5：準備圖示

建立一張 20×20 或 64×64 的 PNG 圖片，存為 `assets/myfirstapp/textures/app/hello.png`。

如果暫時沒有圖示，可以用一張純色圖片代替。

---

## 資料夾結構

| 路徑 | 說明 |
|------|------|
| `META-INF/services/` | SPI 清單，告訴 MCphone 你的 App 類別在哪裡 |
| `com/example/` | Java 原始碼（套件路徑對應資料夾） |
| `assets/myfirstapp/lang/` | 語言檔（zh_cn.json、en_us.json 等） |
| `assets/myfirstapp/textures/app/` | 應用圖示 |

**命名空間**：`myfirstapp` 是你的資源命名空間，建議用小寫英文和數字，不要用大寫或特殊符號。

---

## SPI 清單

SPI（Service Provider Interface）是 Java 的標準機制，讓程式可以自動發現實作特定介面的類別。

MCphone 使用 SPI 來發現所有 App。你的 JAR 裡必須有這個檔案：

```
META-INF/services/com.november.mcphone.api.client.app.IPhoneApp
```

內容是你的 App 類別的完整名稱（套件 + 類別名），每行一個：

```
com.example.HelloApp
com.example.AnotherApp
```

**常見錯誤**：
- ❌ 檔名有副檔名（如 `.txt`）
- ❌ 類別名稱錯誤（大小寫、套件路徑）
- ❌ 類別沒有實作 `IPhoneApp` 介面

---

## 語言檔

MCphone 使用 Minecraft 的標準語言檔機制。語言檔是 JSON 格式，放在 `assets/<命名空間>/lang/` 資料夾。

### 支援的語言

| 檔名 | 語言 |
|------|------|
| `zh_cn.json` | 簡體中文 |
| `zh_tw.json` | 繁體中文 |
| `en_us.json` | 英文（美國） |
| `ja_jp.json` | 日文 |
| `ko_kr.json` | 韓文 |

### 建議的翻譯鍵

```json
{
  "app.<命名空間>.<app_id>": "應用名稱",
  "app.<命名空間>.<app_id>.desc": "應用描述"
}
```

例如：
```json
{
  "app.myfirstapp.hello": "哈囉世界",
  "app.myfirstapp.hello.desc": "我的第一個 MCphone 應用"
}
```

---

## 應用圖示

- 格式：PNG
- 建議尺寸：64×64 或 128×128
- 路徑：`assets/<命名空間>/textures/app/<app_id>.png`

在 Java 程式碼中引用：
```java
@Override
public ResourceLocation getIconTexture() {
    return ResourceLocation.fromNamespaceAndPath(
        "myfirstapp", 
        "textures/app/hello.png"
    );
}
```

---

## 進階：互動式頁面

### 點擊偵測

`IPhonePage` 介面有 `mouseClicked` 方法，可以偵測滑鼠點擊：

```java
public static class MyPage implements IPhonePage {
    
    private int clickCount = 0;
    
    @Override
    public void render(PhoneCanvas canvas) {
        canvas.fill(0, 0, canvas.width, canvas.height, 0xFF1a1a2e);
        canvas.drawCentered("點擊次數：" + clickCount, canvas.width / 2, 40, 0xFFFFFFFF);
        
        // 繪製按鈕
        canvas.fill(20, 80, canvas.width - 40, 40, 0xFFE94560);
        canvas.drawCentered("點我！", canvas.width / 2, 95, 0xFFFFFFFF);
    }
    
    @Override
    public boolean mouseClicked(double mouseX, double mouseY, int button) {
        // 檢查是否點擊了按鈕區域
        if (mouseX >= 20 && mouseX <= canvas.width - 20 && 
            mouseY >= 80 && mouseY <= 120) {
            clickCount++;
            return true; // 消費這個點擊事件
        }
        return false;
    }
}
```

### 鍵盤輸入

如果你的頁面需要鍵盤輸入，覆寫 `capturesKeyboard()` 方法回傳 `true`：

```java
@Override
public boolean capturesKeyboard() {
    return true;
}
```

### 關閉頁面

當頁面被關閉時，`onClose()` 方法會被呼叫，可以在這裡做清理工作：

```java
@Override
public void onClose() {
    // 儲存狀態、釋放資源等
}
```

---

## PhoneCanvas 常用方法

| 方法 | 說明 |
|------|------|
| `fill(x, y, width, height, color)` | 繪製填色矩形 |
| `text(text, x, y, color)` | 繪製文字 |
| `drawCentered(text, x, y, color)` | 繪製置中文字 |
| `border(x, y, width, height, color)` | 繪製邊框 |
| `hline(x, y, length, color)` | 繪製水平線 |
| `vline(x, y, length, color)` | 繪製垂直線 |

**顏色格式**：ARGB 十六進位，例如 `0xFFFFFFFF`（白色）、`0xFFE94560`（紅色）、`0x80000000`（半透明黑色）。

---

## 編譯與打包

### 方法 1：命令列（適合簡單專案）

```bash
# 編譯（需要 mcphone jar 在 classpath）
javac -cp "mcphone-1.9.1.jar" -d build/ com/example/HelloApp.java

# 打包成 JAR
cd build
jar cf my-first-app.jar META-INF/ com/ assets/
```

### 方法 2：Gradle（適合大型專案）

建立 `build.gradle`：

```groovy
plugins {
    id 'java'
}

repositories {
    mavenCentral()
    flatDir { dirs 'libs' }
}

dependencies {
    implementation files('libs/mcphone-1.9.1.jar')
}

jar {
    archiveFileName = 'my-first-app.jar'
    from(sourceSets.main.output)
}
```

然後執行：
```bash
gradle build
```

### 方法 3：手動打包（Windows 用戶）

1. 把所有檔案放進一個資料夾
2. 用 7-Zip 把整個資料夾壓成 ZIP
3. 把副檔名從 `.zip` 改成 `.jar`

**注意**：JAR 檔的根目錄必須直接是 `META-INF/`、`com/`、`assets/`，不能多一層資料夾。

---

## 測試

1. 把編譯好的 JAR 檔放進 `.minecraft/mods/` 資料夾
2. 啟動 Minecraft
3. 拿出手機物品，右鍵打開
4. 在應用列表中找到你的 App
5. 點擊打開，確認功能正常

### 除錯

如果 App 沒有出現：
1. 檢查 SPI 清單檔名是否正確（沒有副檔名）
2. 檢查 SPI 清單內容的類別名稱是否正確
3. 檢查 JAR 檔結構（根目錄必須直接是 META-INF/、com/、assets/）
4. 查看遊戲紀錄檔（`.minecraft/logs/latest.log`）有沒有錯誤訊息

---

## 上傳到資源站

當你的 App 測試完成後，可以上傳到 MCphone Ultra 資源站，讓其他玩家下載：

1. 打開資源站：https://ryanwu047-wq.github.io/mcphone-ultra-store/
2. 點「如何上傳」中的 GitHub 倉庫連結
3. 進入 `apps/` 資料夾
4. 點「Add file」→「Upload files」
5. 把你的 JAR 檔拖進去
6. 點「Commit changes」
7. 完成！其他玩家就可以在資源站下載你的 App 了

### 上傳規範

- 檔案格式：`.jar` 或 `.zip`
- 最大檔案大小：10 MB
- 必須包含有效的 SPI 清單
- 嚴禁包含惡意程式碼、後門
- 建議包含應用圖示和語言檔

---

## 常見問題

### Q：我的 App 沒有出現在手機裡？

A：請檢查：
1. SPI 清單檔名是否正確（`META-INF/services/com.november.mcphone.api.client.app.IPhoneApp`，沒有副檔名）
2. SPI 清單內容的類別名稱是否正確（套件 + 類別名）
3. JAR 檔結構是否正確（根目錄直接是 META-INF/、com/、assets/）
4. 類別是否有實作 `IPhoneApp` 介面的所有必要方法

### Q：遊戲崩潰了怎麼辦？

A：查看 `.minecraft/logs/latest.log`，搜尋你的 App 類別名稱，看錯誤訊息。常見原因：
- 類別路徑錯誤
- 缺少必要的方法實作
- 資源檔路徑錯誤

### Q：可以用第三方函式庫嗎？

A：可以，但需要把第三方函式庫一起打包進 JAR（fat JAR / uber JAR），或者確認玩家已經安裝了該函式庫。建議盡量使用 Minecraft 和 MCphone 內建的功能。

### Q：可以做多個 App 在同一個 JAR 嗎？

A：可以！在 SPI 清單裡每行寫一個類別名稱即可。每個 App 都需要有自己的 `getId()`（不能重複）。

### Q：App 的 ID 有什麼規則？

A：格式是 `<命名空間>:<app_id>`，例如 `myfirstapp:hello`。命名空間和 app_id 都只能用小寫英文、數字、底線和連字號。

---

## 參考資源

- MCphone 原始碼：https://github.com/november521/mcphone
- MCphone Addon API 文件：https://github.com/november521/mcphone/blob/main/docs/addon-api.md
- MCphone Ultra 資源站：https://ryanwu047-wq.github.io/mcphone-ultra-store/
- MCphone Ultra Mod：https://github.com/ryanwu047-wq/mcphone-ultra

---

祝你開發順利！🎉
