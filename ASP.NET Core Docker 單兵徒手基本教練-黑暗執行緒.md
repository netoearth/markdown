<table><tbody><tr><td><img src="https://blog.darkthread.net/img/calendar.svg"></td><td><span title="Published at 2022-09-13 10:05 PM"><time datetime="2022-09-13T14:05:21" itemprop="datePublished">2022-09-13 10:05 PM</time></span></td><td><img src="https://blog.darkthread.net/img/comment.svg"></td><td><span title="0 comments">0</span></td><td><img src="https://blog.darkthread.net/img/eye.svg"></td><td><span data-ajax-url="/blog/pageviewcount/aspnetcore-docker-with-cli" title="2,217 pageviews">2,217</span></td><td><a href="https://blog.darkthread.net/Blog/aspnetcore-docker-with-cli/"><img src="data:image/gif;base64,R0lGODlhAQABAIAAAP///wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==" id="nllbtn"></a></td></tr></tbody></table>

ASP.NET Core 配合 Docker 是我目前自己架網站的主要做法，主流開發工具已有支援，像是 VSCode 有 [Remote - Containers 延伸模組](https://docs.microsoft.com/zh-tw/training/modules/use-docker-container-dev-env-vs-code/3-use-as-development-environment?WT.mc_id=DOP-MVP-37580)，Visual Studio 也[內建 Docker 支援](https://docs.microsoft.com/en-us/dotnet/architecture/containerized-lifecycle/design-develop-containerized-apps/visual-studio-tools-for-docker?WT.mc_id=DOP-MVP-37580)，但我學新東西習慣先嘗試不用工具徒手完成，藉此了解運作原理，之後再用工具省時間，這麼做的好處是工具時能有足夠的知識偵錯與排除。因此，在這篇文章我將用 dotnet、docker CLI 指令實地演練，建立一個以 Docker 運行的 ASP.NET Core 網站，並包含將資料儲存到 Host 檔案系統的常見應用需求。

本次基本教練所需知識大部分可參考 MS Docs 這篇 [Tutorial: Containerize a .NET app](https://docs.microsoft.com/en-us/dotnet/core/docker/build-container?tabs=linux&WT.mc_id=DOP-MVP-37580)，另外我會用到老朋友 - [Docker Compose](https://blog.darkthread.net/blog/aspnetcore-docker-notes-2/) 簡化設定及管理。

開始前，要先安裝好 [.NET SDK](https://dotnet.microsoft.com/en-us/download?WT.mc_id=DOP-MVP-37580) 及 [Docker Desktop](https://www.docker.com/products/docker-desktop/) (個人、教育、開源專案及小企業用的免費版，安裝在 Windows 可直接跑 Linux 容器，有六小時最多下載 200 次 Image 上限)。

以下為操作步驟：

1.  建立 ASP.NET Core 專案 `dotnet new web -o AspNetCoreInDocker`
    
2.  修改 Program.cs，寫一個輸入訊息寫入 logs.txt 並顯示的網頁：
    
    ```
     var builder = WebApplication.CreateBuilder(args);
     var app = builder.Build();
    
     var html = @"
     <!DOCTYPE html>
     <html>
         <head>
             <meta charset=""utf-8"">
             <title>Log</title>
         </head>
         <body>
             <div>
                 <form action=/add method=post target=submitResult>
                     <input placeholder=message name=msg autocomplete=off />
                     <input type=submit value=Add />
                 </form>
             </div>
             <iframe name=submitResult style=""display:none""></iframe>
             <iframe src=/logs style=""margin-top:6px""></iframe>
         </body>
     </html>
     ";
     var dataFilePath = Path.Combine(
         app.Environment.ContentRootPath, "logs.txt");
     if (!File.Exists(dataFilePath))
         File.WriteAllText(dataFilePath, "");
     app.MapGet("/", () => Results.Content(html, "text/html"));
     app.MapPost("/add", (HttpRequest request) =>
     {
         var msg = request.Form["msg"];
         File.AppendAllText(dataFilePath, $"{DateTime.Now:mm:ss} {msg}{Environment.NewLine}");
         return Results.Content(@"<script>
         parent.document.getElementsByTagName('iframe')[1].src=
             '/logs?_=' + new Date().getTime();
         </script>", "text/html");
     });
     app.MapGet("/logs", () => Results.Content(
         File.ReadAllText(dataFilePath), "text/plain"));
     app.Run();
    ```
    
    輸入的訊息會加註時間附加到 logs.txt，為求簡單，我用 iframe 顯示 logs.txt 內容，執行起來會像這樣：  
    ![](https://blog.darkthread.net/Posts/files/Fig1_637986748081202631.gif)
    
3.  接著在專案根目錄新增一個 Dockerfile，其內容如下：
    
    ```
    FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build-env
    WORKDIR /app
    
    # Copy everything
    COPY . ./
    # Restore as distinct layers
    RUN dotnet restore
    # Build and publish a release
    RUN dotnet publish -c Release -o out
    
    # Build runtime image
    FROM mcr.microsoft.com/dotnet/aspnet:6.0
    WORKDIR /app
    COPY --from=build-env /app/out .
    ENTRYPOINT ["dotnet", "AspNetCoreInDocker.dll"]
    ```
    
    Dockerfile 是本次練習的重點，其中定義 Docker Image 的建立步驟：
    
    -   先下載建立內含 .NET 6 SDK 可編譯專案的容器並指定別名 build-env，將工作目錄切換到容器中的 /app
    -   將 Program.cs、AspNetCoreInDocker.csproj 等檔案複製到容器的 ./ (也就是 /app)
    -   dotnet restore 下載 NuGet 套件
    -   dotnet publish 產生要發佈的檔案到 /app/out
    -   下載並建立 .NET 6 純 Runtime 容器，將工作目錄切換到容器中的 /app
    -   將可編譯容器剛才做好的 /app/out 發佈檔案複製到執行容器的 /app 目錄
    -   設定執行程序 \["dotnet", "AspNetCoreInDocker.dll"\]
4.  執行 `docker build -t aspnetcore-in-docker -f Dockerfile .` 建立 Docker Image：  
    ![](https://blog.darkthread.net/Posts/files/Fig2_637986748085068878.png)  
    使用指令 `docker images` 可以查到建立好的 Image：  
    ![](https://blog.darkthread.net/Posts/files/Fig3_637986748086897997.png)
    
5.  再來我打算用 [Docker Compose](https://blog.darkthread.net/blog/aspnetcore-docker-notes-2/) 來執行 Docker 容器。Docker Desktop 已內含 Docker Compose 不需另外安裝，我們在專案目錄建立如下 docker-compose.yml 檔案：
    
    ```
    aspnetcore-in-docker:
      image: aspnetcore-in-docker
      container_name: aspnetcore-in-docker
      ports:
        - 5000:80
      volumes:
        - /x/Github/AspNetCoreInDocker/logs.txt:/app/logs.txt
    ```
    
    網站架構單純，YAML 檔設定也很簡單，指定 Image 名稱、容器名稱，將容器 80 Port 導向本機的 5000 Port，logs.txt 檔則映對到本機檔案(在 Windows 環境下 X:/... 要寫成 /x/...)，資料才能長永保存，不會因容器切換消失。
    
6.  使用 `docker-compose up` 指令依 docker-compose.yml 設定啟動容器，開啟瀏覽器在 ℎttp://localhost:5000 可看到網站執行，結果也有寫入 X:\\Github\\AspNetCoreInDocker\\logs.txt，測試成功!  
    ![](https://blog.darkthread.net/Posts/files/Fig4_637986748089099690.png)
    

練習完畢。

Tutorial of building docker image for ASP.NET Core with CLI tools.