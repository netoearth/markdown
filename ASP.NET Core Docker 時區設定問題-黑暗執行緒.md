<table><tbody><tr><td><img src="https://blog.darkthread.net/img/calendar.svg"></td><td><span title="Published at 2022-09-14 10:00 PM"><time datetime="2022-09-14T14:00:53" itemprop="datePublished">2022-09-14 10:00 PM</time></span></td><td><img src="https://blog.darkthread.net/img/comment.svg"></td><td><span title="0 comments">0</span></td><td><img src="https://blog.darkthread.net/img/eye.svg"></td><td><span data-ajax-url="/blog/pageviewcount/set-timezone-for-aspnetcore-docker" title="578 pageviews">578</span></td><td><a href="https://blog.darkthread.net/Blog/set-timezone-for-aspnetcore-docker/"><img src="data:image/gif;base64,R0lGODlhAQABAIAAAP///wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==" id="nllbtn"></a></td></tr></tbody></table>

昨天介紹的 ASP.NET Core Docker 做法有個小問題 - 若程式有用到本地時間，在本機執行與在 Docker 容器的結果會不同。原因是 Docker Image 的預設時區為 UTC，與本機不同。

用一小段範例重現問題。在 Program.cs 加入一個 timezone-check 動態網頁，顯示系統時區、伺服器端本地時間及瀏覽器本地時間 ：

```
app.MapGet("/timezone-check", () => Results.Content(
        @$"<!DOCTYPE html>
        <html><head><meta charset=""utf-8""></head>
        <body>
        <div>{TimeZoneInfo.Local.StandardName}</div>
        <div>{DateTime.Now.ToLongTimeString()}</div>
        <div id=time></div>
        <script>document.getElementById('time').innerText=new Date().toLocaleTimeString()</script>
        </body></html>", "text/html"));
```

使用 dotnet run 在本機執行，時區為台北標準時間，伺服器與瀏覽器的本地時間一致：

![](https://blog.darkthread.net/Posts/files/Fig1_637987617237998920.png)

當部署到 Docker 容器，會發現容器作業系統的預設時區是 UTC，存在 8 小時時差：

![](https://blog.darkthread.net/Posts/files/Fig2_637987617239861497.png)

最直接了當的解決方法是修改 Dockerfile，將 Docker 容器作業系統時區改成指定時區，做法為加入 `ENV TZ="Asia/Taipei"`(感謝讀者 Chin Khai Hong 分享) 或 `RUN ln -sf /usr/share/zoneinfo/Asia/Taipei /etc/localtime`：(時區名稱，如本例中的「Asia/Taipei」，可查詢 [tz database time zones 清單](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones))

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
ENV TZ="Asia/Taipei"
# RUN ln -sf /usr/share/zoneinfo/Asia/Taipei /etc/localtime
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "AspNetCoreInDocker.dll"]
```

修改後，Docker 容器時區就會改成台北標準時間囉!

![](https://blog.darkthread.net/Posts/files/Fig3_637987617241674135.png)

至於語系國別日期格式，我習慣由 .NET 程式主控，不會跟隨作業系統異動，所以就先不研究如何調整了。收工。

Tips of how to set timezone of docker for ASP.NET Core.