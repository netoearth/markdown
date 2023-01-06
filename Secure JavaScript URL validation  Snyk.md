When developers need to handle URLs in different forms for different purposes — such as browser history navigation, anchored targets, query parameters, and so on — we often turn to JavaScript. However, its frequent use motivates attackers to exploit its vulnerabilities. This risk of exploitation is why we must implement URL validation in our JavaScript applications.

URL validation checks whether URLs follow proper URL syntax — the structure every URL must have. URL validation can save our applications from URL-based vulnerabilities like malicious script injection and server-side request forgery (SSRF). Malicious actors can employ SSRF attacks when we don’t apply secure coding conventions to validate user-supplied URLs when fetching a remote resource. SSRF [remains a critical threat](https://www.acunetix.com/blog/articles/server-side-request-forgery-vulnerability/) to JavaScript-based applications, both for the frontend and Node.js server-side applications, and was a featured category in the [OWASP Top 10](https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/) list in 2021.

## URL validation

URL validation exists to strengthen security against possible exploits and eliminate the chance of any bugs arising while running code. But when should we use URL validation, and what do we validate during the process? We should implement URL validation in all software that must identify and verify resources like pages, images, gifs, and videos. 

A typical URL comprises multiple fragments, such as a protocol, domain name, hostname, resource name, origin, port, and so on. These tell the browser how to retrieve the particular resource. We can use these to validate URLs in several ways:

-   Using Regex literals and constructors
-   URL constructors
-   `isValidURL` method
-   Input elements
-   Anchor tag method

A typical URL validation scheme receives input from a user and then parses it to identify its various components. The validation scheme can ensure that all URL components comply with internet standards. For example, if required, it can check whether the URL uses a secure protocol. 

Hostname validation begins by breaking hostnames into separate labels to ensure they comply with the [Top Level Domain Name Specification](https://www.rfc-editor.org/rfc/rfc1591.txt#:~:text=There%20are%20a%20set%20of,created%20a%20hierarchy%20of%20names.). A typical hostname consists of a minimum of two labels segregated by dots. For example, [www.snyk.com](http://www.snyk.com/) has the labels “www,” “snyk,” and “com”. Each label can only consist of an alphanumeric character or a hyphen, regardless of their case. Then, the validation scheme can ensure the hostname matches an allowlist of allowable URLs to ensure only designated URLs are allowed and allowed URLs are not wrongfully disqualified.

Most paths to a resource used in URLs are allowed by default. However, ports can only be in the range of 1 to 65536. Anything outside that range should throw an error. We can also check numerical IP addresses to gauge if it’s an IPV4 address or an IPV6 address.

Finally, although not obvious initially, we can also check URLs for usernames and passwords. This feature helps with compliance with company policies and credential protection. 

Now that you have the basics, let’s walk through URL validation using javascript!

## How to perform URL validation in JavaScript

The easiest way to perform URL validation in JavaScript is by using the `new URL` constructor function. In addition to its simplicity, it’s supported by the Node.js runtime and [most browsers](https://developer.mozilla.org/en-US/docs/Web/API/URL/URL#browser_compatibility).

The basic syntax is as follows:

```
  new URL (url)
    new URL (url , base)
```

JavaScript only requires the `base` element if we provide a relative URL. If we don’t provide one, it defaults to `undefined`. Alternatively, if we provide a `base` element with an absolute URL, JavaScript ignores the `base` element.

To validate the URL, the following function can be used:

```
function checkUrl (string) {
    let givenURL ;
    try {
        givenURL = new URL (string);
    } catch (error) {
        console.log ("error is", error);
       return false; 
    }
    return true;
  }
```

This function checks the validity of a URL — it returns true when the URL is valid and false when it’s not. If you pass `www.urlcheck.com` to this function, it will return false because it does not contain a valid URL scheme. The correct version of this URL is `https://urlcheck.com`. Another example is `mailto:John.Doe@example.com`. This is a valid URL, but if you remove the colon, JavaScript will no longer recognize it as a URL. A third example is `ftp://`. This is not valid because it doesn’t contain a hostname. If you add two dots (`..`), it will become valid because the dots will be considered a hostname, in this way `ftp://..` becomes a valid URL. 

It’s important to to bear in mind that unconventional, but perfectly valid, URLs exist! They might be unexpected for the developers working on them, but are otherwise perfectly appropriate. For example, both of the following URLs will return TRUE:

-   new URL(“youtube://a.b.c.d”);
-   new URL (“a://1.2.3.4@1.2.3.4”);

These examples are a reminder that developers should rely on the principles of URL validation, instead of focusing on the conventions.

If you want to make sure that the valid URL contains some specific URL scheme, you can use the following function:

```
  function checkHttpUrl(string) {
    let givenURL;
    try {
        givenURL = new URL(string);
    } catch (error) {
        console.log("error is",error)
      return false;  
    }
    return givenURL.protocol === "http:" || givenURL.protocol === "https:";
  }
```

This function validates the URL, and then checks whether or not it’s using the HTTP or HTTPS schemes. Here, `ftp://..` will be invalid because it does not contain HTTP or HTTPS, while `http://..` remains valid.  
Some other ways to use the `URL` constructor function include:

```
  let m = 'https://snyk.io';
  let a = new URL("/", m); 
```

The above example uses the `base` element. Logging the value gives us `https://snyk.io/`.

To return a URL object without specifying the `base` parameter, the syntax is:

```
  let b = new URL(m);
```

To add a pathname to the host, we structure it as follows:

```
  let d = new URL('/en-US/docs', b);
```

The URL stored in `d` is `https://snyk.io/en-US/docs`.

Another functionality of the URL module is that it implements the [WHATWG URL API](https://www.ibm.com/docs/en/datapower-gateway/10.5?topic=module-whatwg-url-api), which adheres to the WHATWG URL standard browsers use:

```
  let adr = new URL("https://snyk.io/en-US/docs");
  let host = adr.host;
  let path = adr.pathname;
```

In the above example, we created a URL object called `adr`. Then, the code fetched the host and the pathname of the URL — which are `snyk.io` and `/en-US/docs`, respectively. Finally, we can compare the URL to an allowlist or blocklist to ensure only designated URLs are allowed and allowed URLs are not wrongfully disqualified.

### How to validate a URL using regex — although you shouldn’t

Another way to validate a URL is by using a regular expression (regex) — or a string that forms a search pattern. We can use Regex to check whether the URL is valid.

The JavaScript syntax for URL validation using regex is:

```
  function isValidURL(string) 
        {
            var res = 
            string.match(/(https?:\/\/(?:www\.|(?!www))[a-zA-Z0-9][a-zA-Z0-9-
            ]+[a-zA-Z0-9]\.[^\s]{2,}|www\.[a-zA-Z0-9][a-zA-Z0-9-]+[a-zA-Z0-9]
            \.[^\s]{2,}|https?:\/\/(?:www\.|(?!www))[a-zA-Z0-9]+\.[^\s]{2,}|w
            ww\.[a-zA-Z0-9]+\.[^\s]{2,})/gi);
        return (res !== null);
        };
```

To test some URLs:

```
  var tc1 = "http://helloworld.com"
  console.log(isValidURL(tc1));
```

The regex-defined URL syntax checks whether the URL starts with the `http://` or `https://` schemes or a subdomain, and whether it contains a domain name. The statement on the console is `true` because it follows the URL syntax defined by the regex. In contrast, the following statement will return a `false` value because it doesn’t start with any of the allowed schemes or a subdomain, nor does it contain a domain name: 

```
  var tc4 = "helloWorld";
  console.log (isValidURL(tc4));
```

The regex above is relatively simple but still difficult to navigate. It’s also an error-prone approach because a regex cannot adequately handle the rules for validating a URL. The most it can do is match valid URLs. Furthermore, executing the validation check becomes time-consuming when a regular expression either contains a complex validation logic or receives a lengthy input string. 

To meet the defined regex validation checks, the browser must backtrack on the order of millions through the input string. This much backchecking could result in “catastrophic backtracking”, a phenomenon in which complex regular expressions can freeze the browser or max out the CPU core processes.

### Real-world vulnerabilities and resolutions

[Node.js](https://nodejs.dev/en/) is a free, open source, cross-platform JavaScript runtime environment that uses a package manager called Node Package Manager (npm). 

A similar event occurred in 2019 when an attacker single-handedly obtained unauthorized access to data from Capital One, one of the biggest banks in the United States. That data breach remains one of the most significant of the century. According to [_The New York Times_](https://www.nytimes.com/2019/07/29/business/capital-one-data-breach-hacked.html), the attacker gained access to 100 million customer records, 140,000 Social Security numbers, and 80,000 linked bank details of Capital One customers. 

The attacker accessed the Capital One server hosted by Amazon Web Services (AWS). The delay in detection indicates that the attacker had a deep knowledge of the AWS infrastructure and saw an exploitable vulnerability in the Managed Security Service Provider’s (MODSEC) Web Application Firewall (WAF). Armed with this knowledge, the perpetrator carried out the SSRF attack, and made new HTTP requests by manipulating the vulnerable web server — thereby successfully gaining access to the AWS metadata service. 

Depending on company requirements, we can avoid SSRF attacks by restricting the system to only a few protocols, such as HTTP or HTTPS. To add more security, we must set up the application so it doesn’t pass the URL parameter without oversight. Allow and deny lists are typically used to filter and control the mechanism, reducing the probability of SSRF significantly. Allow lists enable the application to use a predefined assortment of objects and servers. Meanwhile, deny lists restrict the application from retrieving commonly available hostnames.

## Using JavaScript safely

As evidenced by the addition of SSRF to the new OWASP Top 10, URL validation has become increasingly crucial for JavaScript application security. Fortunately, we can help mitigate such attacks by validating URLs on the server side. Additionally, using the new `URL` function according to the preferred ways to validate and work with URLs can be highly beneficial. 

After seeing a few use cases of the new `URL` function’s capability, we learned how to validate a URL with regex — and saw why this approach is cumbersome and error-prone. We then concluded with a case study of the SSRF JavaScript vulnerability. 

The security risks with URLs are less about their validity and more about dangerous URL schemes. As such, we need to ensure that the server-side application carries out the validation. An attacker can bypass the client-side validation mechanism, so relying solely on that isn’t the way to go.

To learn more about managing your application security, visit [Snyk](https://snyk.io/) today. 

## Find and fix JavaScript vulnerabilities for free

Create a Snyk account today for safe and secure JavaScript applications.