---
title: httpstat – A Curl Statistics Tool to Check Website Performance
date: 2018-05-04T14:48:53+08:00
tags: [golang, httpstat]
---
The httpstat is a tool that reflects curl statistics in a fascinating and well-defined way.


![example](/images/example.png)

## Install

The httpstat has been implemented by golang or python.

### Install by python
- Get it directly from its github repo using the wget commands as follows:
```
wget -c https://raw.githubusercontent.com/reorx/httpstat/master/httpstat.py
```
- Using pip like this
```
sudo -H pip install httpstat
```

### Install by golang
#### Notice
httpstat requires Go 1.7.1 or later

Install it like this
```
go get -u github.com/davecheney/httpstat
```

## How to use
List help info
```
httpstat --help
```
Access watchguard website
```
httpstat www.watchguard.com

Connected to 104.17.60.6:443

HTTP/2.0 200 OK
Server: cloudflare
Cache-Control: public, max-age=1800
Cf-Cache-Status: MISS
Cf-Ray: 415919205adc930c-SJC
Content-Language: en
Content-Type: text/html; charset=utf-8
Date: Fri, 04 May 2018 07:01:48 GMT
Expect-Ct: max-age=604800, report-uri="https://report-uri.cloudflare.com/cdn-cgi/beacon/expect-ct"
Expires: Sun, 19 Nov 1978 05:00:00 GMT
Last-Modified: Fri, 04 May 2018 06:56:19 GMT
Link: <https://www.watchguard.com>; rel="canonical",<https://www.watchguard.com/>; rel="shortlink"
Set-Cookie: __cfduid=dd1ef17a31a0b0aacf4df8cdbfe1d38ec1525417308; expires=Sat, 04-May-19 07:01:48 GMT; path=/; domain=.watchguard.com; HttpOnly,BIGipServerwatchguard.com-p443=2348340278.47873.0000; expires=Fri, 04-May-2018 19:00:56 GMT; path=/; Httponly; Secure
Vary: Cookie,Accept-Encoding
Via: 1.1 varnish-v4
X-Ah-Environment: prod
X-Cache: HIT
X-Cache-Hits: 105
X-Drupal-Cache: HIT
X-Frame-Options: SAMEORIGIN
X-Request-Id: v-5a0ca084-4f68-11e8-8ffd-22000a6746ba
X-Varnish: 16916405 16916090

Body discarded

  DNS Lookup   TCP Connection   TLS Handshake   Server Processing   Content Transfer
[      9ms  |         183ms  |        438ms  |            324ms  |           189ms  ]
            |                |               |                   |                  |
   namelookup:9ms            |               |                   |                  |
                       connect:192ms         |                   |                  |
                                   pretransfer:630ms             |                  |
                                                     starttransfer:955ms            |
                                                                                total:1144ms
```

## Demo
Now, we would create a demo to verify the Server Processing time is accurately or not. We would create a http server, listened to a port, sleep some time. In the end, verify the time what show by httpstat

```
package main

import (
	"net/http"
	"strings"
	"time"
)

func sayHello(w http.ResponseWriter, r *http.Request) {
	message := r.URL.Path
	message = strings.TrimPrefix(message, "/")
	message = "Hello " + message
	time.Sleep(2 * time.Second)
	w.Write([]byte(message))
}
func main() {
	http.HandleFunc("/", sayHello)
	if err := http.ListenAndServe(":8080", nil); err != nil {
		panic(err)
	}
}
```
Run the code, it would listened to 8080 port, and sleep 2 seconds.

Accsss the http service using httpstat
```
httpstat http://127.0.0.1:8080

Connected to 127.0.0.1:8080

HTTP/1.1 200 OK
Content-Length: 6
Content-Type: text/plain; charset=utf-8
Date: Fri, 04 May 2018 07:18:42 GMT

Body discarded

   DNS Lookup   TCP Connection   Server Processing   Content Transfer
[       0ms  |           4ms  |           2002ms  |             0ms  ]
             |                |                   |                  |
    namelookup:0ms            |                   |                  |
                        connect:4ms               |                  |
                                      starttransfer:2006ms           |
                                                                 total:2007ms
```

You can see the Server Processing time is 2002ms.

## Use it in golang source code
We can use other golang package "github.com/tcnksm/go-httpstat", it can show the Server Processing in golang code.

Example is here
```
package main

import (
	"io"
	"io/ioutil"
	"log"
	"net/http"
	"time"

	httpstat "github.com/tcnksm/go-httpstat"
)

func main() {
	req, err := http.NewRequest("GET", "http://127.0.0.1:8080", nil)
	if err != nil {
		log.Fatal(err)
	}
	// Create a httpstat powered context
	var result httpstat.Result
	ctx := httpstat.WithHTTPStat(req.Context(), &result)
	req = req.WithContext(ctx)
	// Send request by default HTTP client
	client := http.DefaultClient
	res, err := client.Do(req)
	if err != nil {
		log.Fatal(err)
	}
	if _, err := io.Copy(ioutil.Discard, res.Body); err != nil {
		log.Fatal(err)
	}
	res.Body.Close()
	// end := time.Now()
	// Show the results
	log.Printf("DNS lookup: %d ms", int(result.DNSLookup/time.Millisecond))
	log.Printf("TCP connection: %d ms", int(result.TCPConnection/time.Millisecond))
	log.Printf("TLS handshake: %d ms", int(result.TLSHandshake/time.Millisecond))
	log.Printf("Server processing: %d ms", int(result.ServerProcessing/time.Millisecond))
	log.Printf("Content transfer: %d ms", int(result.ContentTransfer(time.Now())/time.Millisecond))
}

```
Run it, you can see this.
```
go run httpstatExample.go
2018/05/04 15:27:58 DNS lookup: 0 ms
2018/05/04 15:27:58 TCP connection: 1 ms
2018/05/04 15:27:58 TLS handshake: 0 ms
2018/05/04 15:27:58 Server processing: 2001 ms
2018/05/04 15:27:58 Content transfer: 8 ms
```
