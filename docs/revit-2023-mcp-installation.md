# Revit 2023 MCP 安裝與 SDK 開發教學

本指南分成兩條路線：

- **想直接在 Revit 2023 使用 MCP**：安裝已打包的 Revit 外掛，並連接 Claude Desktop。
- **想開發自己的 Revit 外掛**：在 C# 專案引用 RevitMCPSDK NuGet 套件，再自行實作外掛入口與 MCP 傳輸連線。

> RevitMCPSDK 是開發用程式庫，不是可直接安裝的 Revit 外掛或 MCP 用戶端伺服器。若只想開始使用，請走第一條路線。

## 路線 A：安裝可直接使用的 Revit MCP

本節採用 [LuDattilo/revit-mcp-server](https://github.com/LuDattilo/revit-mcp-server) 的預先建置外掛。該專案的快速入門列出 Revit 2023 支援，並提供 Revit 2023 專用 ZIP。這是第三方專案，並非 Autodesk 或 RevitMCPSDK 官方發行品。

### 需求

- Windows 電腦上已安裝 Autodesk Revit 2023。
- 已安裝並登入 Claude Desktop。
- 可從 GitHub Releases 下載安裝 ZIP。

### 安裝外掛

1. 開啟 [Revit MCP Releases](https://github.com/LuDattilo/revit-mcp-server/releases)，下載檔名包含 `Revit2023` 的 ZIP。指南編寫時，最新列出的版本為 `v2.2.1`；下載時請以 Releases 頁面目前提供的版本為準。
2. 按 `Win + R`，輸入以下路徑後按 Enter：

   ```text
   %AppData%\Autodesk\Revit\Addins\2023
   ```

   若資料夾不存在，先建立 `2023` 資料夾。
3. 將 ZIP 內容直接解壓到該資料夾。解壓後應能看到：

   ```text
   Addins\2023\
   ├── mcp-servers-for-revit.addin
   └── revit_mcp_plugin\
       ├── RevitMCPPlugin.dll
       └── Commands\
   ```

   `.addin` 檔必須直接位於 `Addins\2023`，不能多包一層解壓資料夾。
4. 關閉並重新開啟 Revit 2023。若 Revit 顯示外掛載入提示，確認載入。
5. 在 Revit 的 **Add-Ins** 頁籤找到 **Revit MCP Switch**、**MCP Panel** 和 **Settings**。按 **Revit MCP Switch** 啟動連線，確認狀態為綠色。
6. 完全結束 Claude Desktop（包括系統匣中的程序）後重新開啟。在對話輸入框附近確認 Revit MCP 工具已出現。

依該專案快速入門，外掛首次啟動會嘗試自動設定 Claude Desktop。若沒有自動完成，請先查看專案的 [Quick Start](https://github.com/LuDattilo/revit-mcp-server/blob/main/QUICK_START.md) 與 [Troubleshooting](https://github.com/LuDattilo/revit-mcp-server/blob/main/QUICK_START.md#troubleshooting)，不要把不同 MCP 專案的設定範例混在一起。

### 初次連線與試用

1. 在 Revit 開啟一個可用的測試模型。
2. 按 **Revit MCP Switch** 啟動連線。
3. 重新啟動 Claude Desktop，確認工具清單出現。
4. 先用唯讀問題確認連線，例如「列出目前模型的樓層」。
5. 確認結果正確後，再嘗試建立或修改元素。

大批修改前請另存測試模型，並先要求 Claude 說明將要執行的變更。此第三方專案包含可執行 Revit 程式碼的工具；只在你信任該程式碼的情況下安裝及使用。

### 移除外掛

1. 關閉 Revit。
2. 刪除 `%AppData%\Autodesk\Revit\Addins\2023\mcp-servers-for-revit.addin` 與同層的 `revit_mcp_plugin` 資料夾。
3. 若 MCP 工具仍出現在 Claude Desktop，移除該專案新增的 Claude Desktop 設定，再重新啟動 Claude Desktop。

### 常見問題

| 狀況 | 檢查方式 |
| --- | --- |
| Revit 沒有顯示 MCP 按鈕 | 確認下載的是 Revit 2023 ZIP，且 `.addin` 在 `Addins\2023` 根目錄。 |
| Windows 阻擋 DLL | 在 Windows 檔案內容檢查是否有「解除封鎖」選項；只對從可信來源取得的檔案解除封鎖。 |
| Claude Desktop 看不到工具 | 確認 Revit MCP Switch 已啟動，再完全結束並重開 Claude Desktop。 |
| 連線逾時 | 確認 Revit 已開啟模型且外掛狀態為綠色；重新啟動 Revit 與 Claude Desktop。 |
| Revit 載入失敗 | 查看 Revit 外掛載入提示及該專案的問題排除說明；不要安裝其他 Revit 年份的 ZIP。 |

## 路線 B：在 Revit 2023 外掛專案使用 RevitMCPSDK

這條路線提供給 C# 開發者。Revit 2023 的 API 專案使用 **.NET Framework 4.8**。請先安裝 Visual Studio（含 .NET 桌面開發工作負載）與 Revit 2023。

### 安裝 NuGet 套件

在 Visual Studio 建立或開啟 Revit 外掛的 C# Class Library 專案，將目標 Framework 設為 `.NET Framework 4.8`，再使用 NuGet Package Manager Console 執行：

```powershell
Install-Package RevitMCPSDK -Version 2023.0.0.4
```

也可以在專案檔加入：

```xml
<PackageReference Include="RevitMCPSDK" Version="2023.0.0.4" />
```

NuGet 套件：[RevitMCPSDK 2023.0.0.4](https://www.nuget.org/packages/RevitMCPSDK/2023.0.0.4)。若版本已更新，請查看 NuGet 的 Revit 2023 套件版本頁面，並使用相容版本。

### 建置與載入自己的外掛

1. 依照 Revit API 的開發方式建立自己的 `IExternalApplication` 或 `IExternalCommand`，在外掛中呼叫 SDK 的命令與模型類別。
2. 建置專案，將需要的 DLL 複製到外掛目錄。
3. 建立 Revit `.addin` manifest，令其指向建置出的 DLL 與外掛入口類別。開發階段可放在：

   ```text
   %AppData%\Autodesk\Revit\Addins\2023
   ```

4. 關閉並重新開啟 Revit 2023，確認外掛載入，再以測試模型驗證功能。
5. 若要讓 AI 用戶端透過 MCP 呼叫外掛，還需實作並設定 MCP 伺服器／傳輸連線。**安裝 RevitMCPSDK NuGet 套件不會自動提供這部分，也不會產生安裝器。**

RevitMCPSDK 的 [README](https://github.com/chanweaa/RevitMCPSDK#readme) 含有命令基底類別與註冊範例；Autodesk 的 [Revit API 開發需求](https://help.autodesk.com/cloudhelp/2024/CHS/Revit-API/files/Revit_API_Developers_Guide/Introduction/Getting_Started/Welcome_to_the_Revit_Platform_API/Revit_API_Revit_API_Developers_Guide_Introduction_Getting_Started_Welcome_to_the_Revit_Platform_API_Development_Requirements_html.html) 說明 API 所需的 .NET Framework 及 Revit API DLL 參考。

## 參考連結

- [Revit MCP Server Releases](https://github.com/LuDattilo/revit-mcp-server/releases)
- [Revit MCP Quick Start](https://github.com/LuDattilo/revit-mcp-server/blob/main/QUICK_START.md)
- [RevitMCPSDK NuGet 套件](https://www.nuget.org/packages/RevitMCPSDK/2023.0.0.4)
- [RevitMCPSDK 原始碼](https://github.com/chanweaa/RevitMCPSDK)
- [Autodesk Revit API 開發需求](https://help.autodesk.com/cloudhelp/2024/CHS/Revit-API/files/Revit_API_Developers_Guide/Introduction/Getting_Started/Welcome_to_the_Revit_Platform_API/Revit_API_Revit_API_Developers_Guide_Introduction_Getting_Started_Welcome_to_the_Revit_Platform_API_Development_Requirements_html.html)
