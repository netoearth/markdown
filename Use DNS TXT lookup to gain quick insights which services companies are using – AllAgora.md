Modern web services consist of a variety of different own and third-party services connected and integrated to form a seamless experience (in the best case). And this happens for a good reason: As companies focus on their core business, they outsource everything that is not of strategic value for their operations such as ticketing systems, marketing & email platform and the like. But with high heterogeneity comes high integration effort and one of the standard methods to integrate services from different providers in an orderly and clean way is to put them under your own domain.  
Simply put, if you have a site **example.com** and you use an IT support ticketing system, you may want to make it reachable over **itsupport.example.com** while the payment processing system may live under **payment.example.com**. As mentioned, these services are often third-party cloud SaaS applications that are created and registered under the domain name of the respective provider, the ticketing solution may be provided by **ticketingsystem.com** and when developers of example.com register a service, they may choose a subdomain like **examplecompany.ticketingsystem.com**. To make examplecompany.ticketingsystem.com reachable over itsupport.example.com, a CNAME entry is created in the DNS servers that resolve example.com which may look like this:

<table><tbody><tr><td>NAME</td><td>TTL</td><td>VALUE</td></tr><tr><td>itsupport</td><td>600</td><td>examplecompany.ticketingsystem.com</td></tr></tbody></table>

When a user wants to access itsupport.example.com, a DNS request is sent to the nameservers of example.com which hold a copy of the table above. The servers see that the user wants to access the “itsupport” subdomain and therefore sends back a DNS response containing the value examplecompany.ticketingsystem.com. The client would continue to resolve this new name until it gets the IP address of the target service it should contact.

Now here is the catch: ticketingsystem.com doesn’t know that example.com is the eligible domain to forward requests to examplecompany.ticketingsystem.com. But especially in certain areas, this is critical to know, think of a white hacking service which deliberately attacks example.com to find security risks – in that case the service certainly wants to verify that only owners of example.com start the process and this step is called **domain ownership verification**. In the most common verification approach, ticketingsystem.com creates a random string for each customer like “ticketsystem-verify=a7027db21c” and if example.com wants to redirect any subdomain to ticketsystem.com, it has to prove first that it has ownership over the example.com domain. This happens by creating a TXT record at the DNS server that resolves example.com which contains the random string:

<table><tbody><tr><td>NAME</td><td>TTL</td><td>VALUE</td></tr><tr><td></td><td>600</td><td>ticketsystem-verify=a7027db21c</td></tr></tbody></table>

Now someone like newcompany.com could create a CNAME entry **itsupport.newcompany.com** pointing to **examplecompany.ticketingsystem.com**. If ticketingsystem.com wants to prevent that, it simply checks if there is a TXT record at newcompany.com that contains the value “ticketsystem-verify=a7027db21c” – if not not, it denies to deliver the requested site. And since the random string is only presented to the creator of examplecompany.ticketingsystem.com, newcompany.com doesn’t know it and therefore can’t set it in their DNS server. Usually this check happens only once at the verification step and not on every request so once ticketingsystem.com knows that the correct TXT record exists, the verification is done and the site is delivered.

However, a lot of companies keep their TXT records in their DNS servers forever. And since a lot of services nowadays use this method, it allows us to gain interesting statistical insights into which domains use certain services. On Linux systems, a tool called “dig” is available (in dnsutils or similar utility packages) that allow to query DNS servers for all kinds of records. For example, to check the TXT records for salesforce.com, you simply run

```
dig -t txt salesforce.com
```

which returns

```
;; QUESTION SECTION:
;salesforce.com.                        IN      TXT

;; ANSWER SECTION:
IN      TXT     "google-site-verification=D6BlHxqITDdvcLDrxA3"
IN      TXT     "fastly-domain-delegation-lololo98765-341965-2021-07-14"
IN      TXT     "atlassian-domain-verification=BbMCUfMM/IdZKdn891dRUlB8hY0"
IN      TXT     "docker-verification=b238c187-0eb6-4710-ac1f-0d2ed19765b5"
IN      TXT     "v=spf1 include:_spf.google.com include:_spf.salesforce.com"
IN      TXT     "stripe-verification=9ca4e73f9b48dbec579fe0"
IN      TXT     "google-site-verificqZHxzvbEU_LFMiy4XwYtJiSS1BhQ"
IN      TXT     "pardot1=6eae4d5ab8dffb7553ea905"
IN      TXT     "pardot220122=922f8d6c3558cebb481d367d6f006"
IN      TXT     "google-site-verification=HV79FO1Y0siBF9WSte-fAOzLI3om9c1M"
IN      TXT     "google-site-verification=XHgruaJj29eI7YjqDkE_QM"
IN      TXT     "SFMC-cGJQFeEomoQQt-tQ3c_QXefdwzOGuj_Tjl4oWwrW"
```

Without much effort we now know that Salesforce is using GCP, Stripe, Fastly, Atlassian and other services. When using a service like [nslookup.io](https://www.nslookup.io/) you even get a nice looking interface that presents these TXT records:

[![](https://allagora.files.wordpress.com/2022/03/image.png?w=1024)](https://allagora.files.wordpress.com/2022/03/image.png)

Even though these “long-living domain verification TXT records” are not a direct security risk, they can be used to get insights about companies. Especially if you track the TXT records for several companies over a long period of time, you gain knowledge when a company starts using a service.