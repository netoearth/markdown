> 👉️**URL:** [https://caddyserver.com/docs/automatic-https](https://caddyserver.com/docs/automatic-https)
> 
> 📝**Description:**
> 
> Caddy 是一个强大的、企业级的、开源的 web 服务器，使用 Go 自动编写 HTTPS

**Caddy 是一个强大的、企业级的、开源的 web 服务器，使用 Go 自动编写 HTTPS**

自动化 HTTPS 规定所有站点的 TLS 证书并 **保持更新** 。它还为您重定向 HTTP 到 HTTPS ! Caddy 使用安全和现代的 **默认设置**——不需要停机、额外的配置、或单独的工具。

下面是一段 28 秒的视频，展示了它是如何工作的：

<iframe width="100%" height="480" src="https://www.youtube-nocookie.com/embed/nk4EWHvvZtI?rel=0" frameborder="0" allowfullscreen=""></iframe>

## 概述[](https://ewhisper.cn/posts/20225/#%E6%A6%82%E8%BF%B0%20-14)

**默认情况下，Caddy 通过 HTTPS 服务所有网站**

-   Caddy 通过 HTTPS 提供 IP 地址和本地 / 内部主机名，使用本地自动信任的自签名证书（如果允许）。
    -   实例：`localhost`, `127.0.0.1`
-   Caddy 通过 HTTPS 服务公共 DNS 名称，使用来自公共 ACME CA 的证书，如 [Let’s Encrypt](https://letsencrypt.org/) 或 [ZeroSSL](https://zerossl.com/)。
    -   示例：`example.com`, `sub.example.com`, `*.example.com`

Caddy 保持所有管理的证书 **更新**，并自动将 HTTP（默认端口 80) 重定向到 HTTPS（默认端口 443)。

**对于本地 HTTPS:**

-   Caddy 可能会提示输入密码，以便将其惟一的根证书安装到您的信任存储区中。这在每个根中只发生一次；你可以在任何时候移除它。
-   任何不信任 Caddy root 而访问该站点的客户端都将显示安全错误。

**对于公共域名**

-   如果您的域的 A/AAAA 记录指向您的服务器，
-   80、443 端口对外开放，
-   Caddy 可以绑定那些端口 (_或_ 那些端口转发给 Caddy)，
-   你的 [数据目录](https://caddyserver.com/docs/conventions#data-directory) 是可写的和持久的，
-   和你的域名出现在某处相关的配置，

然后网站将自动通过 HTTPS 服务。你不需要再做什么了。它就是正常工作了！

由于 HTTPS 使用共享的公共基础设施，作为服务器管理员的您应该理解此页面上的其余信息，以便您可以避免不必要的问题，在出现问题时排除故障，并正确配置高级部署。

## 激活[](https://ewhisper.cn/posts/20225/#%E6%BF%80%E6%B4%BB)

当 Caddy 知道服务的域名（即主机名）或 IP 地址时，它会隐式地自动激活 HTTPS。有多种方式告诉 Caddy 您的域名 /IP，这取决于您如何运行或配置 Caddy:

-   [Caddyfile](https://caddyserver.com/docs/caddyfile) 中的一个 [站点地址](https://caddyserver.com/docs/caddyfile/concepts#addresses)
-   [route](https://caddyserver.com/docs/modules/http#servers/routes) 中的一条 [host matcher](https://caddyserver.com/docs/json/apps/http/servers/routes/match/host/)
-   命令行 flags 如 [–domain](https://caddyserver.com/docs/command-line#caddy-file-server) 或 [–from](https://caddyserver.com/docs/command-line#caddy-reverse-proxy)
-   [automate](https://caddyserver.com/docs/json/apps/tls/certificates/automate/) 证书 loader

以下任何一项将阻止自动 HTTPS 被激活，无论是全部或部分：

-   [显示禁用它](https://caddyserver.com/docs/json/apps/http/servers/automatic_https/)
-   在配置中不提供任何主机名或 IP 地址
-   只监听 HTTP 端口
-   手动加载证书（除非 [this config property](https://caddyserver.com/docs/json/apps/http/servers/automatic_https/ignore_loaded_certificates/) 为真）

## 效果[](https://ewhisper.cn/posts/20225/#%E6%95%88%E6%9E%9C)

当 HTTPS 协议自动激活时，会出现以下情况：

-   [所有域名](https://caddyserver.com/docs/automatic-https#hostname-requirements) 获得证书并更新
-   默认端口（如果有）改为 [HTTPS 端口](https://caddyserver.com/docs/modules/http#https_port) 443
-   HTTP 被重定向到 HTTPS（使用 [HTTP 端口](https://caddyserver.com/docs/modules/http#http_port) 80)

自动 HTTPS 永远不会覆盖显式的配置。

如果需要，您可以 [自定义或禁用自动 HTTPS](https://caddyserver.com/docs/json/apps/http/servers/automatic_https/); 例如，你可以跳过某些域名或禁用重定向（对于 Caddyfile，使用 [global options](https://caddyserver.com/docs/caddyfile/options))。

## Hostname 需求[](https://ewhisper.cn/posts/20225/#Hostname-%20%E9%9C%80%E6%B1%82)

所有主机名（域名）符合完全管理证书的条件，如果它们：

-   是非空的
-   仅由字母数字、连字符、点和通配符 (`*`) 组成
-   不要以点开头或结尾 ([RFC 1034](https://tools.ietf.org/html/rfc1034#section-3.5))

此外，主机名符合公共信任证书的条件：

-   不是 localhost （包括`.localhost` 和`.local` 等）
-   不是 IP 地址
-   只有一个通配符 `*` 作为最左边的标签

为了通过 HTTPS 服务于非公共站点，Caddy 生成自己的证书颁发机构 (CA)，并使用 CA 来签署证书。信任链由根证书和中间证书组成。叶子证书是由中间体签名的。它们存储在 [Caddy 的数据目录](https://caddyserver.com/docs/conventions#data-directory) 的`pki/authorities/local` 。

Caddy 的本地 CA 是由 [Smallstep 库](https://smallstep.com/certificates/) 支持的。

本地 HTTPS 不使用 ACME，也不执行任何 DNS 验证。它只在本地机器上工作，并且只在安装 CA 的根证书的地方受信任。

### CA Root[](https://ewhisper.cn/posts/20225/#CA-Root)

根的私钥是使用加密安全的伪随机源 (cryptographically-secure pseudorandom source) 惟一生成的，并以有限的权限保存到存储中。它被加载到内存中只是为了执行签名任务，然后它将在随后执行垃圾收集。

虽然可以将 Caddy 配置为直接使用根签名（以支持不兼容的客户端），但这在默认情况下是禁用的，根密钥仅用于为中间体签名。

第一次使用根密钥时，Caddy 将尝试将其安装到系统的本地信任存储中。如果它没有这样做的权限，它将提示输入密码。如果不需要，可以在配置中禁用此行为。

在安装了 Caddy 的根 CA 之后，您将在本地信任存储区中看到它作为“Caddy local Authority”（除非您配置了不同的名称）。您可以随时卸载它 ([`caddy untrust`](https://caddyserver.com/docs/command-line#caddy-untrust) 命令使这很容易）。

请注意，将证书自动安装到本地信任存储区只是为了方便，并不能保证能正常工作，特别是在使用容器或将 Caddy 作为非特权系统服务运行时。最终，如果您依赖于内部 PKI，则系统管理员有责任确保将 Caddy 的根 CA 正确地添加到必要的信任存储库中（这超出了 web 服务器的范围）。

### CA Intermediates[](https://ewhisper.cn/posts/20225/#CA-Intermediates)

还将生成一个中间证书和密钥，用于签署页（leaf ）（单个站点）证书。

与根证书不同，中间证书的生存期要短得多，并且会根据需要自动更新。

## 测试[](https://ewhisper.cn/posts/20225/#%E6%B5%8B%E8%AF%95)

要测试或实验你的 Caddy 配置，确保你 \[修改 ACME 端点\]([https://caddyserver.com/docs/modules/tls.issuance.acme](https://caddyserver.com/docs/modules/tls.issuance.acme) ca) 到一个 staging 或开发环境的 URL, 否则你可能会遇到速率限制，这可能会阻止你访问 HTTPS 长达一周，这取决于你遇到的速率限制。

Caddy 的默认 ca 之一是 [Let’s Encrypt](https://letsencrypt.org/)，它有一个 [staging 端点](https://letsencrypt.org/docs/staging-environment/) ，不受相同 [速率限制](https://letsencrypt.org/docs/rate-limits/)) 的限制：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code><span>https:</span>//acme-staging-v02.api.letsencrypt<span>.org</span>/directory<br></code><p><i></i>AVRASM</p></pre></td></tr></tbody></table>

## ACME challenges[](https://ewhisper.cn/posts/20225/#ACME-challenges)

获取受公众信任的 TLS 证书需要从受公众信任的第三方权威机构进行验证。现在，这个验证过程是通过 [ACME 协议](https://tools.ietf.org/html/rfc8555) 自动进行的，可以通过以下三种方式 (“challenge 类型”(challenge types)) 中的一种来执行。

默认情况下，前两种 challenge types 是启用的。如果启用了多个 challenges，Caddy 会随机选择一个 challenge ，以避免偶然依赖于特定的 challenge 。随着时间的推移，它会了解哪种 challenge 类型最成功，并首先开始倾向于它，但如果有必要，它会返回到其他可用的 challenge 类型。

### HTTP challenge[](https://ewhisper.cn/posts/20225/#HTTP-challenge)

HTTP challenge 对候选主机名的 A/AAAA 记录执行权威 DNS 查找，然后使用 HTTP 请求端口 80 上的临时加密资源。如果 CA 看到预期的资源，则颁发证书。

此 challenge 要求从外部访问端口 80。如果 80 端口无法监听，则 80 端口的报文必须转发到 Caddy 的 [HTTP 端口](https://caddyserver.com/docs/json/apps/http/http_port/)。

这个 challenge 默认情况下是启用的，不需要显式的配置。

### TLS-ALPN challenge[](https://ewhisper.cn/posts/20225/#TLS-ALPN-challenge)

TLS-ALPN challenge 对候选主机名的 A/AAAA 记录执行权威 DNS 查找，然后使用包含特殊 ServerName 和 ALPN 值的 TLS 握手通过端口 443 请求临时加密资源。如果 CA 看到预期的资源，则颁发证书。

此 challenge 需要外部可访问端口 443。如果 443 端口无法监听，则需要将来自 443 端口的报文转发给 Caddy 的 [HTTPS 端口](https://caddyserver.com/docs/json/apps/http/https_port/)。

这个 challenge 默认情况下是启用的，不需要显式的配置。

### DNS challenge[](https://ewhisper.cn/posts/20225/#DNS-challenge)

DNS challenge 对候选主机名的 TXT 记录执行权威性的 DNS 查找，并查找具有特定值的特殊 TXT 记录。如果 CA 看到了期望的值，则颁发证书。

此 challenge 不需要任何开放端口，请求证书的服务器不需要从外部访问。然而，DNS challenge 需要配置。Caddy 需要知道访问您的域的 DNS 提供商的凭据，以便它可以设置（和清除）特殊的 TXT 记录。启用 DNS challenge 功能后，其他 challenge 功能默认关闭。

DNS 提供者支持是社区的努力。[在我们的维基上了解如何为您的提供商启用 DNSchallenge](https://caddy.community/t/how-to-use-dns-provider-modules-in-caddy-2/8148)

## 按需 TLS（On-Demand TLS）[](https://ewhisper.cn/posts/20225/#%E6%8C%89%E9%9C%80%20-TLS%EF%BC%88On-Demand-TLS%EF%BC%89)

Caddy 开创了一种我们称为 **On-Demand TLS** 的新技术，它在第一次需要 TLS 握手时动态地获得新证书，而不是在配置加载时。重要的是，这并不需要在您的配置中提前指定域名。

许多企业依靠这个独特的特性，以更低的成本扩展 TLS 部署，并且在服务数万个站点时不会遇到运营方面的麻烦。

On-demand TLS 使用场景：

-   当你启动或重新加载服务器时，你不知道所有的域名，
-   域名可能没有正确配置 (DNS 记录还没有设置），
-   你不能控制域名（例如，他们是客户的域名）。

当启用按需 TLS 时，您不需要在配置中指定域名来获得证书。相反，当接收到服务器名 (SNI) 的 TLS 握手时，Caddy 还没有证书，握手将保持，同时 Caddy 获得证书来完成握手。延迟通常只有几秒钟，而且只有最初的握手是缓慢的。所有以后的握手都是快速的，因为证书被缓存和重用，并且更新是在后台进行的。以后的握手可能会触发对证书的维护，以保持更新，但是如果证书还没有过期，这种维护就会在后台发生。

### 使用 On-Demand TLS[](https://ewhisper.cn/posts/20225/#%E4%BD%BF%E7%94%A8%20-On-Demand-TLS)

**在生产环境中，按需 TLS 必须同时启用和进行限制。启用而不限制会使服务器受到攻击**

启用按需 TLS 发生在 [TLS 自动化策略](https://caddyserver.com/docs/json/apps/tls/automation/policies/) ，如果使用 JSON 配置，或如果使用 Caddyfile 的话 [在站点块与’ TLS ' 指令](https://caddyserver.com/docs/caddyfile/directives/tls) 这里。

为了防止此特性被滥用，必须配置限制。这是在 JSON 配置的 [’ automation '对象](https://caddyserver.com/docs/json/apps/tls/automation/on_demand/)，或 Caddyfile 的 [’ on\_demand\_tls' 全局选项](https://caddyserver.com/docs/caddyfile/options#on-demand-tls) 中完成的。限制是“全局的”，不能对每个站点或域进行配置。主要的限制是一个“询问 (ask)”端点，Caddy 将向这个端点发送一个 HTTP 请求，询问它是否有权限获取和管理握手中域的证书。这意味着您将需要一些内部后端，例如，可以查询数据库的帐户表，并查看客户是否使用该域名注册。

您还可以将速率限制配置为限制 (configure rate limits as restrictions)，尽管速率限制本身并不能提供足够的保护。

请注意您的 CA 能够多快地颁发证书。如果花费的时间超过几秒钟，这将对用户体验产生负面影响（仅针对第一个客户端）。

由于它的延迟性和滥用的可能性（如果没有通过适当的配置来缓解），我们建议只有在上面描述了实际用例时才启用按需 TLS。

[有关有效使用按需 TLS 的更多信息，请参阅我们的 wiki 文章](https://caddy.community/t/serving-tens-of-thousands-of-domains-over-https-with-caddy/11179)

## 错误[](https://ewhisper.cn/posts/20225/#%E9%94%99%E8%AF%AF)

如果证书管理出现错误，Caddy 会尽力继续。

缺省情况下，证书管理在后台进行。这意味着它不会阻止启动或减慢您的网站。然而，这也意味着服务器将在所有证书可用之前运行。在后台运行允许 Caddy 在很长一段时间内以频繁 回退（backoff） 并重试。

以下是获取或更新证书时发生的错误：

1.  Caddy 重试一次后，一个简短的暂停，只是以防万一，它是一个侥幸（fluke）
2.  Caddy 短暂停顿，然后切换到下一个启用的 challenge 类型
3.  在尝试了所有启用的 challenge 类型之后，[它尝试下一个配置的发行商](https://caddyserver.com/docs/automatic-https#issuer-fallback)
    -   Let’s Encrypt
    -   ZeroSSL
4.  在所有发行商都尝试过之后，它会以指数方式退出
    -   尝试期间最多尝试 1 天
    -   尝试总时间长达 30 天

在使用 Let’s Encrypt 进行重试时，Caddy 切换到他们的 [staging 环境](https://letsencrypt.org/docs/staging-environment/) 以避免速率限制问题。这不是一个完美的策略，但总的来说是有帮助的。

ACME challenge 至少需要几秒钟，内部速率限制有助于减少意外滥用。除了您或 CA 配置的内容外，Caddy 还使用内部速率限制，以便您可以将一个包含 100 万个域名的大盘子 (platter) 交给 Caddy，并且它将逐渐地——但尽可能快地——获得所有这些域名的证书。Caddy 的内部速率限制目前是每 10 秒对每个 ACME 帐户做 10 次尝试。

为了避免资源泄漏，当配置被更改时，Caddy 会中止运行中的任务（包括 ACME 事务）。虽然 Caddy 能够处理频繁的配置重新加载，但请注意诸如此类的操作考虑，并考虑批处理配置更改以减少重新加载，并让 Caddy 有机会在后台实际完成证书的获取。

### Issuer 回退[](https://ewhisper.cn/posts/20225/#Issuer-%20%E5%9B%9E%E9%80%80)

在无法成功获得证书的情况下，Caddy 是第一个（也是迄今为止唯一一个）支持完全冗余、自动故障转移到其他 ca 的服务器。

默认情况下，Caddy 启用两个 acme 兼容的 CAs: [**Let’s Encrypt**](https://letsencrypt.org/) 和 [**ZeroSSL**](https://zerossl.com/)。如果 Caddy 不能从 Let’s Encrypt 获得证书，它将尝试使用 ZeroSSL; 如果两者都失败，它将回退并稍后重试。在您的配置中，您可以自定义 Caddy 使用哪些颁发者来获得证书（通用的或特定的名称）。

## 存储[](https://ewhisper.cn/posts/20225/#%E5%AD%98%E5%82%A8%20-2)

Caddy 将在它的 [已配置的存储设施](https://caddyserver.com/docs/json/storage/) 中存储公共证书、私钥和其他资产（如果没有配置，也可以是默认的——详情请参阅链接）。

**使用默认配置你需要知道的主要事情是 `$HOME` 文件夹必须是可写和持久的**. 如果`--environ` 标志被指定，为了帮助您排除故障，Caddy 在启动时会打印它的环境变量。

任何配置为使用相同存储的 Caddy 实例将自动共享这些资源，并作为集群协调证书管理。

在尝试任何 ACME 事务之前，Caddy 将测试配置的存储，以确保它是可写的，并且有足够的容量。这有助于减少不必要的锁争用。

## 通配符证书[](https://ewhisper.cn/posts/20225/#%E9%80%9A%E9%85%8D%E7%AC%A6%E8%AF%81%E4%B9%A6)

当将 Caddy 配置为提供具有合格通配符名称的站点时，它可以获取和管理通配符证书。如果站点名称只有最左边的域标签是通配符，则该站点名称有资格使用通配符。例如， `*.example.com` 是满足限定条件的，但这些是不行的：`sub.*.example.com`, `foo*.example.com`, `*bar.example.com`, 和`*.*.example.com`.

如果使用 Caddyfile，那么 cadyfile 将根据证书主题名称获取站点名称。换句话说，定义为 `sub.example.com` 的站点将导致 Caddy 管理 `sub.example.com` 的证书，而定义为 `*.example.com` 的站点将导致 Caddy 管理 `*.example.com` 的通配符证书。您可以在我们的 [Common Caddyfile Patterns](https://caddyserver.com/docs/caddyfile/patterns#wildcard-certificates) 页面上看到演示。如果需要不同的行为，[JSON 配置](https://caddyserver.com/docs/json/) 可以对证书主题和站点名称 (“主机匹配器” host matchers) 进行更精确的控制。

通配符证书代表了广泛的权限，只有当您有很多子域，以至于管理单个证书会使 PKI 紧张或使您达到 ca 强制的速率限制时才应该使用通配符证书。

**注：** [Let’s Encrypt 需要](https://letsencrypt.org/docs/challenge-types/) [DNS challenge](https://caddyserver.com/docs/automatic-https#dns-challenge) 来获取通配符证书。

## 额外参考资料[](https://ewhisper.cn/posts/20225/#%E9%A2%9D%E5%A4%96%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99)

[Using Caddy to keep certificates renewed - Wiki - Caddy Community](https://caddy.community/t/using-caddy-to-keep-certificates-renewed/7525)