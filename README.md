# How to Set up And Change User Agent with cURL

[![Bright Data Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/)

This guide walks you through the process of configuring and modifying the User-Agent header in cURL to improve your web scraping efforts:

- [User Agents and Their Significance](#user-agents-and-their-significance)
- [The Standard cURL User Agent](#the-standard-curl-user-agent)
- [Methods to Configure the cURL User Agent Header](#methods-to-configure-the-curl-user-agent-header)
  - [Directly Setting a Custom User Agent](#directly-setting-a-custom-user-agent)
  - [Configuring a Custom User Agent Header](#configuring-a-custom-user-agent-header)
- [Creating a User Agent Rotation System in cURL](#creating-a-user-agent-rotation-system-in-curl)
  - [Implementation in Bash](#implementation-in-bash)
  - [Implementation in PowerShell](#implementation-in-powershell)
- [Final Thoughts](#final-thoughts)

## User Agents and Their Significance

A user agent represents a text string that web browsers, request-making applications, and HTTP clients include in the [User-Agent](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent) HTTP header to reveal the source software of the request. This identifier typically contains details about the browser/application type, operating system, and additional specifications.

Here's an example of a typical user agent string from a Chrome browser:

```
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36
```

This header information plays a crucial role in determining if requests originate from legitimate browsers or other software tools. Automated scraping tools often utilize inconsistent or generic user agent strings, revealing their automated nature. Therefore, the user agent header assists anti-bot technologies in distinguishing between genuine users and automated scripts.

## The Standard cURL User Agent

Similar to most HTTP clients, [cURL](https://curl.se/) automatically includes a User-Agent header when sending HTTP requests. The default cURL user agent follows this format:

```
curl/X.Y.Z 
```

Where X.Y.Z represents your installed cURL version.

To confirm this, send a test request to the /user-agent endpoint provided by [httpbin.io](https://httpbin.io/), which returns the User-Agent header sent by the requester:

```sh
curl "https://httpbin.io/user-agent"
```

> **Note**:
> Windows users should use curl.exe instead of curl. This distinction exists because curl serves as an alias for [Invoke-WebRequest](https://brightdata.com/blog/how-tos/powershell-invoke-webrequest-with-proxy) in PowerShell, while curl.exe directs to the actual cURL Windows executable.

The response should appear similar to:

```json
{
  "user-agent": "curl/8.4.0"
}
```

As demonstrated, cURL sets its version (curl/8.4.0) as the user agent. This presents a problem because it explicitly identifies your request as originating from cURL. Website protection mechanisms designed to safeguard content can easily flag such requests as non-human, potentially blocking them.

That's why it's important to customize the cURL user agent header.

## How to Configure the cURL User Agent Header

You can employ two different approaches to modify a user agent in cURL.

### Directly Setting a Custom User Agent

cURL provides a dedicated option for this purpose. Specifically, the [\-A or –user-agent](https://curl.se/docs/manpage.html#-A) flag allows you to define the content of the User-Agent header in cURL requests.

The syntax for establishing a cURL user agent header using this option is:

```sh
curl -A|--user-agent "<user-agent_string>" "<url>"
```

Here is an example:

```sh
curl -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36" "https://httpbin.io/user-agent"
```

For the command above, you'll receive this output:

```json
{
  "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36"
}
```

The command above is equivalent to using the longer form:

```sh
curl --user-agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36" "https://httpbin.io/user-agent"
```

Though not generally advised, you can completely remove the User-Agent header by passing an empty string to `-A`. Verify this by accessing the [/headers](https://httpbin.io/headers) endpoint from httpbin.io, which displays all headers in the incoming request:

```sh
curl -A "" "https://httpbin.io/headers"
```

The result will show:

```json
{
  "headers": {
    "Accept": [
      "*/*"
    ],
    "Host": [
      "httpbin.io"
    ]
  }
}
```

As anticipated, no User-Agent header appears.

If you prefer to maintain the User-Agent header but with an empty value, provide a single space to `-A`:

```sh
curl -A " " "https://httpbin.io/headers"
```

This time, you'll receive:

```json
{
  "headers": {
    "Accept": [
      "*/*"
    ],
    "Host": [
      "httpbin.io"
    ],
    "User-Agent": [
      ""
    ]
  }
}
```

In that case, the User-Agent header exists but contains an empty value.

### Configuring a Custom User Agent Header

Fundamentally, User-Agent functions as a standard HTTP header. This means you can configure it like any other HTTP header in cURL using the [\-H or –header](https://curl.se/docs/manpage.html#-H) [option](https://curl.se/docs/manpage.html#-H):

```sh
curl -H|--header "User-Agent: <user-agent_string>" "<url>"
```

For a more detailed explanation, check our tutorial on [how to send HTTP headers with cURL](https://brightdata.com/blog/how-tos/http-headers-with-curl).

Here's a demonstration of the `-H` option:

```sh
curl -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36" "https://httpbin.io/user-agent"
```

> **Note**:
> Since [HTTP headers are designed to be case-insensitive](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers), "User-Agent" and "user-agent" are treated identically.

The output will be:

```json
{
  "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36"
}
```

This cURL command works identically to:

```sh
curl --header "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36" "https://httpbin.io/user-agent"
```

For specialized scenarios, consider these two alternative header formats:

- `"User-Agent:"` to eliminate the User-Agent header entirely
- `"User-Agent: "` to include the User-Agent header with an empty value

## Creating a User Agent Rotation System in cURL

When performing automated requests at scale with cURL, using a single User-Agent value may prove insufficient. This is because [anti-bot systems](https://brightdata.com/webinar/bot-detection) track all incoming connections. When they detect numerous requests with identical headers from the same IP address, they're likely to implement restrictions.

The trick is to randomize what IPs the requests are coming from. Rotating user agents helps create the appearance of requests coming from various browsers, reducing the risk of triggering temporary restrictions or blocks.

there are three steps to implement cURL user agent rotation:

1. **Gather user agents**: Compile a collection of authentic user agent strings from diverse browsers, spanning different versions and devices.
2. **Create rotation logic**: Develop code that randomly selects a user agent from your collection.
3. **Randomize your requests**: Apply the selected user agent to each cURL request.

This implementation requires slight code variations between Unix-based systems and Windows. Let's explore both approaches!

### Implementation in Bash

First, collect legitimate user agents from resources like [User Agent String.com](https://useragentstring.com/pages/useragentstring.php) and store them in an array:

```
user_agents=(

    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36"

    "Mozilla/5.0 (Macintosh; Intel Mac OS X 14.5; rv:126.0) Gecko/20100101 Firefox/126.0"

    # Add more user agents here

    "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:126.0) Gecko/20100101 Firefox/126.0"

)
```

Next, create a function that randomly selects from this list using the [RANDOM](https://tldp.org/LDP/abs/html/randomvar.html) variable:

```sh
get_random_user_agent() {

    # number of user agents in the list

    local count=${#user_agents[@]}

    # generate a RANDOM number from 0 to count

    local index=$((RANDOM % count))

    # return the randomly extracted user agent string

    echo "${user_agents[$index]}"

}
```

Now invoke the function, retrieve a randomized user agent, and incorporate it into your cURL request:

```sh
# get the random user agent

user_agent=$(get_random_user_agent)

# perform a cURL request to the given URL

# using the random user agent

curl -A "$user_agent" "https://httpbin.io/user-agent"
```

> **Note**:
> Adjust the target URL to match your specific requirements.

Combining these elements creates this complete bash script:

```sh
#!/bin/bash

# list of user agent strings

user_agents=(

    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36"

    "Mozilla/5.0 (Macintosh; Intel Mac OS X 14.5; rv:126.0) Gecko/20100101 Firefox/126.0"

    # ...

    "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:126.0) Gecko/20100101 Firefox/126.0"

)

get_random_user_agent() {

    # number of user agents in the list

    local count=${#user_agents[@]}

    # generate a RANDOM number from 0 to count

    local index=$((RANDOM % count))

    # return the randomly extracted user agent string

    echo "${user_agents[$index]}"

}

# get the random user agent

user_agent=$(get_random_user_agent)

# perform a cURL request to the given URL

# using the random user agent

curl -A "$user_agent" "https://httpbin.io/user-agent"
```

Run this script multiple times, and you'll notice a different user agent appears each time—mission accomplished!

### Implementation in PowerShell

Begin by gathering authentic user agents from sources like [WhatIsMyBrowser.com](https://www.whatismybrowser.com/). Then store them in an [array variable](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_arrays?view=powershell-7.4):

```sh
$user_agents = @(

    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36"

    "Mozilla/5.0 (Macintosh; Intel Mac OS X 14.5; rv:126.0) Gecko/20100101 Firefox/126.0"

    # Add more user agents here

    "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:126.0) Gecko/20100101 Firefox/126.0"

)
```

Next, develop a function that randomly selects a user agent from your collection:

```sh
function Get-RandomUserAgent {

    # Count the total user agents available

    $count = $user_agents.Count

    # Generate a random index within the valid range

    $index = Get-Random -Maximum $count

    # Return the randomly chosen user agent

    return $user_agents[$index]

}
```

Finally, call this function, obtain the random user agent, and apply it to your cURL request:

```sh
# get the random user agent

$user_agent = Get-RandomUserAgent

# make an HTTP request to the given URL 

# using the random user agent

curl.exe -A "$user_agent" "https://httpbin.io/user-agent"
```

Bringing everything together produces this complete PowerShell script:

```sh
# list of user agents

$user_agents = @(

    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36"

    "Mozilla/5.0 (Macintosh; Intel Mac OS X 14.5; rv:126.0) Gecko/20100101 Firefox/126.0"

    # ...

    "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:126.0) Gecko/20100101 Firefox/126.0"

)

function Get-RandomUserAgent {

    # number of user agents in the list

    $count = $user_agents.Count

    # generate a random number from 0 to $count

    $index = Get-Random -Maximum $count

    # return the randomly extracted user agent

    return $user_agents[$index]

}

# get a random user agent

$user_agent = Get-RandomUserAgent

# make an HTTP request to the given URL 

# using the random user agent

curl.exe -A "$user_agent" "https://httpbin.io/user-agent"
```

Save this as a .ps1 file and execute it several times. Each execution will display a different user agent string.

## Final Thoughts

Customizing the User-Agent header in cURL can help deceive basic anti-bot systems by making your requests appear to come from standard browsers. However, sophisticated anti-bot technologies can still identify and block such requests.

To overcome limitations like rate restrictions, consider [integrating proxies with cURL](https://brightdata.com/blog/proxy-101/curl-with-proxies). For even more robust solutions, explore [Scraper API](https://brightdata.com/products/web-scraper)—a comprehensive, next-generation scraping API designed for automated web requests using cURL or other HTTP clients.

Sign up today for a free trial and experience the difference!