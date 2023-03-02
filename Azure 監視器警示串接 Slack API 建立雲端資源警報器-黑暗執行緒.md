前幾天保哥分享了一則[鬼故事](https://blog.miniasp.com/post/2022/12/27/Azure-Active-Directory-Enable-MFA-Correctly) - 因為沒有正確啟動 MFA (多因素認證)，Azure 登入帳號被駭客入侵，對方偷建了上百台虛擬機器(VM)，幸好觸及消費限制帳號被停用才控制損失。但事件在隔天收到停用通知才爆發，身為常疑神疑鬼的被害妄想體質(例如：沒事會[寫工具查查誰在偷連我的 Windows](https://blog.darkthread.net/blog/ps-list-logon-events/)、[自製 Windows 遠端存取警報器](https://blog.darkthread.net/blog/pktmon-ps-alarm/)，明顯非正常人呀!)，我興起在 Azure 裝警報器的想法，萬一真有人盜用帳號建立 VM 或操作雲端資源時，能即時發送通知以便在第一時間檢查及處理。

即時通報管道我打算共用已在其他系統運行的 [Slack](https://blog.darkthread.net/blog/slack-api/)，我的手機跟 PC 都裝了 Slack 客戶端，加上有發送程式範例可參考，統一用它接收通報最省事。如何在建立 Azure VM 或其他資源時主動通知則是這次要研究的議題，但很快便找到解決方案。

Azure 有所謂的[監視器警示](https://learn.microsoft.com/zh-tw/azure/azure-monitor/alerts/alerts-overview?WT.mc_id=DOP-MVP-37580)功能，可以設定條件監控特定資源的計量數據或 Log， 當吻合預設條件(例如：CPU > 80% 或發特某個特別事件)時透過電子郵件、SMS 和推播通知、Webhook、Azure 函式... 等多種管道發送警示：

![](https://blog.darkthread.net/img/loading.svg)

稍微研究了一下，Azure 上所有的資源操作都會產生活動記錄(建立資源群組、開 VM、建對外 IP... 只要跟資源項目有關都會產生一到多筆活動記錄，甚至連啟動或停止 VM 也有)，只要建一條警示規則，在有活動記錄時發送通知，最基本的示警功能就有了：

![](https://blog.darkthread.net/img/loading.svg)

建立警示規則的做法[官方文件有說明](https://learn.microsoft.com/zh-tw/azure/azure-monitor/alerts/alerts-create-new-alert-rule?tabs=metric&WT.mc_id=DOP-MVP-37580)，這裡不多贅述，要監測所有資源異動，我選擇的條件是 All Administrative operations 事件且狀態為成功者：

![](https://blog.darkthread.net/img/loading.svg)

符合條件要如何通知由「動作群組」決定，最簡單的做法是寄 Email，這樣的話不用額外寫程式就能接到通知，是最簡單的做法。不過我想做得細緻一點並加上客製化(換個說法是愛折騰喜歡自找麻煩啦)，則可選 Webhook，它允許你提供一個 URL 接受 JSON 格式的通知資料，再自行寫程式決定後續處理方式：

![](https://blog.darkthread.net/img/loading.svg)

要知道傳送的 JSON 資料格式，可透過上圖的「測試動作群組」試傳一筆範例資料進行分析。我先寫了蒐集程式接收並轉存，匯出結果如下：

```
{
  "schemaId": "azureMonitorCommonAlertSchema",
  "data": {
    "essentials": {
      "alertId": "/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/test-RG/providers/microsoft.insights/activityLogAlerts/Test_Alert",
      "alertRule": "test-activityLogAlertRule",
      "severity": "Sev4",
      "signalType": "Activity Log",
      "monitorCondition": "Fired",
      "monitoringService": "Activity Log - Administrative",
      "alertTargetIDs": [
        "/subscriptions/11111111-1111-1111-1111-111111111111/resourcegroups/test-RG/providers/microsoft.compute/virtualmachines/test-VM"
      ],
      "configurationItems": [
        "test-VM"
      ],
      "originAlertId": "bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb_123456789012345678901234567890ab",
      "firedDateTime": "2022-12-31T06:01:56.523Z",
      "description": "Alert rule description",
      "essentialsVersion": "1.0",
      "alertContextVersion": "1.0"
    },
    "alertContext": {
      "authorization": {
        "action": "Microsoft.Compute/virtualMachines/restart/action",
        "scope": "/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/test-RG/providers/Microsoft.Compute/virtualMachines/test-VM"
      },
      "channels": "Operation",
      "claims": "{}",
      "caller": "user-email@domain.com",
      "correlationId": "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
      "eventSource": "Administrative",
      "eventTimestamp": "2022-12-31T06:01:56.523Z",
      "eventDataId": "bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb",
      "level": "Informational",
      "operationName": "Microsoft.Compute/virtualMachines/restart/action",
      "operationId": "cccccccc-cccc-cccc-cccc-cccccccccccc",
      "properties": {
        "eventCategory": "Administrative",
        "entity": "/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/test-RG/providers/Microsoft.Compute/virtualMachines/test-VM",
        "message": "Microsoft.Compute/virtualMachines/restart/action",
        "hierarchy": "22222222-2222-2222-2222-222222222222/CnAIOrchestrationServicePublicCorpprod/33333333-3333-3333-3333-3333333333333/44444444-4444-4444-4444-444444444444/55555555-5555-5555-5555-555555555555/11111111-1111-1111-1111-111111111111"
      },
      "status": "Succeeded",
      "subStatus": "",
      "submissionTimestamp": "2022-12-31T06:01:56.523Z",
      "Activity Log Event Description": ""
    }
  }
}
```

格式還算單純，由於我的目的是要知道「有什麼資源被異動」，故只需要由 data.alertContext 的 authorization.action 看動作、由 authorization.scope 看異動對象、從 caller 看操作者，這樣就夠了。依據這個想法，寫幾行 Minimal API 拼裝一下，一個收 JSON 轉 Slack 訊息的迷你服務便完成了：

```
using System.Text.Json.Nodes;
using SlackAPI;

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

const string tokenKey = "BotUserOAuthToken";
const string channelIdKey = "ChannelId";
var botOauthToken = app.Configuration[tokenKey];
if (string.IsNullOrEmpty(botOauthToken))
{
    throw new Exception($"{tokenKey} is not set in the configuration.");
}
var channelId = app.Configuration["ChannelName"];
if (string.IsNullOrEmpty(channelIdKey))
{
    throw new Exception($"{channelIdKey} is not set in the configuration.");
}
var client = new SlackTaskClient(botOauthToken);

app.Map("/", () => "Hi There");

app.MapPost("/alert", async (JsonObject alertMsg) =>
{
    var alert = alertMsg["data"]?["alertContext"];
    var action = alert?["authorization"]?["action"]?.AsValue().GetValue<string>() ?? "unknown";
    var scope = alert?["authorization"]?["scope"]?.AsValue().GetValue<string>() ?? "unknown";
    var caller = alert?["caller"]?.AsValue().GetValue<string>() ?? "unknown";
    await client.PostMessageAsync(channelId, $"Azure resource changed by {caller}]: \n{action}\n{scope}");
    return Results.Ok();
});

app.Run();
```

啟動警示規則後，我啟動並停止一台 VM，並建立新資源群組開新 VM，成功在 Slack 頻道接到通報。(由於警示靠定期排程派送，會有幾分鐘的延遲，但不至影響即時性。)

![](https://blog.darkthread.net/img/loading.svg)

就這樣，一個簡易版的 Azure 資源異動通報就完成了，具有基本的防盜警報功能。而監視器警示 + Webhook 串 Slack/LINE/Discord 的概念也可應用在偵錯系統異常、網站負載過高... 等即時通知，玩出更多有趣的應用。

Example of using Azure alert webhook to send alerts to Slack.