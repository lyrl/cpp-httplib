cpp-httplib
===========

A C++11 header-only HTTP library.

[The Boost Software License 1.0](http://www.boost.org/LICENSE_1_0.txt)

It's extremely easy to setup. Just include **httplib.h** file in your code!

Inspired by [Sinatra](http://www.sinatrarb.com/) and [express](https://github.com/visionmedia/express).

Server Example
--------------

```c++
#include <httplib.h>

int main(void)
{
    using namespace httplib;

    Server svr;

    svr.get("/hi", [](const Request& req, Response& res) {
        res.set_content("Hello World!", "text/plain");
    });

    svr.get(R"(/numbers/(\d+))", [&](const Request& req, Response& res) {
        auto numbers = req.matches[1];
        res.set_content(numbers, "text/plain");
    });

    svr.listen("localhost", 1234);
}
```

### Method Chain

```cpp
svr.get("/get", [](const auto& req, auto& res) {
        res.set_content("get", "text/plain");
    })
    .post("/post", [](const auto& req, auto& res) {
        res.set_content(req.body(), "text/plain");
    })
    .listen("localhost", 1234);
```

### Static File Server

```cpp
svr.set_base_dir("./www");
```

### Logging

```cpp
svr.set_logger([](const auto& req, const auto& res) {
    your_logger(req, res);
});
```

### Error Handler

```cpp
svr.set_error_handler([](const auto& req, auto& res) {
    const char* fmt = "<p>Error Status: <span style='color:red;'>%d</span></p>";
    char buf[BUFSIZ];
    snprintf(buf, sizeof(buf), fmt, res.status);
    res.set_content(buf, "text/html");
});
```

### 'multipart/form-data' POST data

```cpp
svr.post("/multipart", [&](const auto& req, auto& res) {
    auto size = req.files.size();
    auto ret = req.has_file("name1"));
    const auto& file = req.get_file_value("name1");
    // file.filename;
    // file.content_type;
    auto body = req.body.substr(file.offset, file.length));
})
```

Client Example
--------------

### GET

```c++
#include <httplib.h>
#include <iostream>

int main(void)
{
    httplib::Client cli("localhost", 1234);

    auto res = cli.get("/hi");
    if (res && res->status == 200) {
        std::cout << res->body << std::endl;
    }
}
```

### POST

```c++
res = cli.post("/post", "text", "text/plain");
res = cli.post("/person", "name=john1&note=coder", "application/x-www-form-urlencoded");
```

### POST with parameters

```c++
httplib::Map params;
params["name"] = "john";
params["note"] = "coder";
auto res = cli.post("/post", params);
```

### With Progress Callback

```cpp
httplib::Client client(url, port);

// prints: 0 / 000 bytes => 50% complete
std::shared_ptr<httplib::Response> res =
    cli.get("/", [](uint64_t len, uint64_t total) {
        printf("%lld / %lld bytes => %d%% complete\n",
            len, total,
            (int)((len/total)*100));
    }
);
```

![progress](https://user-images.githubusercontent.com/236374/33138910-495c4ecc-cf86-11e7-8693-2fc6d09615c4.gif)

This feature was contributed by [underscorediscovery](https://github.com/yhirose/cpp-httplib/pull/23).

### Range (HTTP/1.1)

```cpp
httplib::Client cli("httpbin.org", 80, httplib::HttpVersion::v1_1);

// 'Range: bytes=1-10'
httplib::Headers headers = { httplib::make_range_header(1, 10) };

auto res = cli.get("/range/32", headers);
// res->status should be 206.
// res->body should be "bcdefghijk".
```

OpenSSL Support
---------------

SSL support is available with `CPPHTTPLIB_OPENSSL_SUPPORT`. `libssl` and `libcrypto` should be linked.

```c++
#define CPPHTTPLIB_OPENSSL_SUPPORT

SSLServer svr("./cert.pem", "./key.pem");

SSLClient cli("localhost", 8080);
```

Copyright (c) 2017 Yuji Hirose. All rights reserved.
