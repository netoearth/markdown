.NET 網站專案建立時會附帶兩個 appsettings 檔：appsettings.json 及 appsettings.Development.json。

ASP.NET Core 執行時會由 appsettings.json 讀取設定，開發測試階段還會載入 appsettings.Development.json，正式部署時則是載入 appsettings.Production.json。如此，我們可以讓同一設定值具有多個版本，所有環境都相同的通用設定放在 appsettings.json，開發測試專用設定放在 appsettings.Development.json、正式環境用的放在 appsettings.Production.json，甚至再區分 Staging、Beta、Preview 等自訂環境。

ASP.NET Core 預設有三種環境模式：Development、Staging 及 Production，並可依不同環境設定不同行為(例如：在 Development 環境才顯示詳細錯誤訊息)，透過 ASPNETCORE\_ENVIRONMENT 環境變數，還可以再增加自訂的環境名稱。而依據執行環境載入不同 appsettings.json 設定，也是 ASP.NET Core 環境的主要應用情境。[參考](https://learn.microsoft.com/zh-tw/aspnet/core/fundamentals/environments?view=aspnetcore-7.0&WT.mc_id=DOP-MVP-37580#environments)

以下來實地演練一下。

先開一個 ASP.NET Core Minimal API 網站專案，除了原有 appsettings.json 及 appsettings.Development.json，我再多建一個 appsettings.Production.json。

![](https://blog.darkthread.net/Posts/files/Fig1_638078294992012825.png)

三個 JSON 檔內容如下。appsettings.json：

```
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "MySetting": "From appsettings.json"
}
```

appsettings.Development.json：

```
{
  "MySetting": "From appsettings.Development.json"
}
```

appsettings.Production.json：

```
{
  "MySetting": "From appsettings.Production.json"
}
```

Program.cs 稍做修改，MapGet("/") 時顯示環境變數 ASPNETCORE\_ENVIRONMENT 及 IConfiguration "MySetting" 內容：

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => @$"
Environment: {Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT")}
MySetting: {app.Configuration["MySetting"]}");

app.Run();
```

使用 dotnet run 執行，連上網站看到的 ASPNETCORE\_ENVIRONMENT 變數為 Development、MySetting 來自 appsettings.Development.json：

![](https://blog.darkthread.net/img/loading.svg)

這是因為在 Properties\\launchSettings.json 中宣告了 ASPNETCORE\_ENVIRONMENT = Development：

![](https://blog.darkthread.net/img/loading.svg)

接著我們 dotnet publish -c Release 發佈網站並掛載到 IIS：

![](https://blog.darkthread.net/img/loading.svg)

實測可發現，如[文件](https://learn.microsoft.com/zh-tw/aspnet/core/fundamentals/environments?view=aspnetcore-7.0&WT.mc_id=DOP-MVP-37580#environments)所說，當未設定 DOTNET\_ENVIRONMENT 或 ASPNETCORE\_ENVIRONMENT，預設為 Production 環境，因此 MySetting 來自 appsettings.Production.json：

![](https://blog.darkthread.net/img/loading.svg)

若要切換成其他環境(這裡以 Development 為例)，可加上 EnvironmentName 參數以 `dotnet publish -c Release -p:EnvironmentName=Development` 發佈檔案，差別會在輸出 web.config 時會加上含 system.webServer/aspNetCore/environmentVariables/environmentVairable name="ASPNETCORE\_ENVIRONMENT"：

![](https://blog.darkthread.net/img/loading.svg)

如此，ASP.NET Core 網站將切換成 Development 並改讀取 appsettings.Development.json：

![](https://blog.darkthread.net/img/loading.svg)

若是用 Visual Studio 的 Publish 功能發佈，可在 FolderProfile.pubxml 加入`<EnvironmentName>Development</EnvironmentName>`達到相同效果。

![](https://blog.darkthread.net/img/loading.svg)

演練完畢。

Example of how to read different appsetting value by ASP.NET Core environment.