## [](https://dev.to/kevwan/best-practices-on-developing-monolithic-services-in-go-3c95#why-to-write-this-best-practices)Why to write this best practices

-   For many startups, we should focus more on delivering application early, and at this time the user base is small and the `QPS` is very low, we should use a simpler technical architecture to accelerate the delivery of the application, and this is where the advantages of monoliths come into play.
-   As I often mentioned in my presentations, while we use monoliths to deliver applications quickly, we also need to consider the future possibilities when developing applications, and we can clearly split modules inside the monolith.
-   Many devs asked what's the best practices on developing monoliths with `go-zero`.

As `go-zero` is a widely used microservices framework, which I have precipitated during the complete development of several large projects. We have fully considered the scenario of monolithic service development.

The monolithic architecture using `go-zero`, as shown in the figure, can also support a large volume of business scale, where `Service` is a monolithic service of multiple `Pods`.

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--f1-h5NRa--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://oscimg.oschina.net/oscnet/up-bc7b235b058a13e6477f67c47b241108bea.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--f1-h5NRa--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://oscimg.oschina.net/oscnet/up-bc7b235b058a13e6477f67c47b241108bea.png)

I'll share in detail how to use `go-zero` to quickly develop a monolithic service with multiple modules.

## [](https://dev.to/kevwan/best-practices-on-developing-monolithic-services-in-go-3c95#monolithic-example)Monolithic example

Let's use an upload and download monolithic service to explain the best practices of `go-zero` monolithic service development, why use such an example?

-   The `go-zero` community often asks how to define `API` files for uploading files and then use `goctl` to generate the code automatically. When I first saw this kind of questions, I thought it was strange, why not use a service like `OSS`? I found many scenarios where the user needs to upload excel files, and then the server discards the file after parsing it. One is that the file is small, and the second is that the service is not serving a large amount of users, so we don't need to bring in `OSS`, which I think is quite reasonable.
    
-   The `go-zero` community also asking how to download files by defining an `API` file and then `goctl` automatically generating it. The reason why such questions are asked through Go is that there are generally two reasons: one is that the business is just starting, so it's easier to lay out a service to get all things done; the other is that I hope to take the advantages of `go-zero`'s built-in `JWT` authentication.
    

This is just an example, no need to go deeper into whether uploading and downloading should be written in `Go`. So let's see how we can solve such a monolithic service, which we call the file service, with `go-zero`. The architecture is as follows.

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--7GekhOYj--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://oscimg.oschina.net/oscnet/up-5a51e42aedfa181b325935bf71741cf0c4b.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--7GekhOYj--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://oscimg.oschina.net/oscnet/up-5a51e42aedfa181b325935bf71741cf0c4b.png)

## [](https://dev.to/kevwan/best-practices-on-developing-monolithic-services-in-go-3c95#monolithic-implementation)Monolithic implementation

### [](https://dev.to/kevwan/best-practices-on-developing-monolithic-services-in-go-3c95#-raw-api-endraw-definition)`API` definition

Devs who have used `go-zero` know that we provide an `API` format file to describe the `RESTful API`, and then we can generate the corresponding code by `goctl` with one shot, and we only need to fill in the corresponding business logic in the `logic` file. Let's see how the `download` and `upload` services define the `API`.

### [](https://dev.to/kevwan/best-practices-on-developing-monolithic-services-in-go-3c95#-raw-download-endraw-api-definition)`Download` API definition

The sample requirement is as follows.

-   Download a file named `<filename>` through the `/static/<filename>` path
-   Just return the content of the file directly

We create a file named `download.api` in the `api` directory with the following content.  

```
syntax = "v1"

type DownloadRequest {
  File string `path: "file"`
}

service file-api {
  @handler DownloadHandler
  get /static/:file(DownloadRequest)
}
```

Enter fullscreen mode Exit fullscreen mode

The syntax of `zero-api` is relatively self-explanatory and means the following.

1.  `syntax = "v1"` means that this is the `v1` syntax of `zero-api`
2.  `type DownloadRequest` defines the request format for `Download`
3.  `service file-api` defines the request route for `Download`

### [](https://dev.to/kevwan/best-practices-on-developing-monolithic-services-in-go-3c95#-raw-upload-endraw-api-definition)`Upload` API definition

The sample requirement is as follows.

-   Upload a file via the `/upload` path
-   Return the upload status via `json`, where `code` can be used to express a richer scenario than `HTTP code`

We create a file called `upload.api` in the `api` directory with the following content.  

```
syntax = "v1"

type UploadResponse {
  Code int `json: "code"`
}

service file-api {
  @handler UploadHandler
  post /upload returns (UploadResponse)
}
```

Enter fullscreen mode Exit fullscreen mode

Explain as follows.

1.  `syntax = "v1"` means this is the `zero-api` `v1` syntax
2.  `type UploadResponse` defines the return format of `Upload`
3.  `service file-api` defines the request route for `Upload`

### [](https://dev.to/kevwan/best-practices-on-developing-monolithic-services-in-go-3c95#here-comes-the-problem)Here comes the problem

We have defined the `Download` and `Upload` services, but how can we put them into a service?

I don't know if you have noticed some details.

1.  either `Download` or `Upload`, we prefixed the `request` and `response` data definition, and did not use directly such as `Request` or `Response`
2.  we define `service` in `download.api` and `upload.api` with `file-api` as the `service name`, not `download-api` and `upload-api` respectively

The purpose of this is to automatically generate the corresponding `Go` code when we put the two services into the monolithic service. Let's see how to merge `Download` and `Upload` together~

### [](https://dev.to/kevwan/best-practices-on-developing-monolithic-services-in-go-3c95#defining-the-monolithic-service-raw-api-endraw-)Defining the monolithic service `API`

For simplicity reasons, `goctl` only supports accepting a single `API` file as a parameter, the issue of accepting multiple `API` files is not discussed here and may be supported later if we figure out a simple and efficient solution.

We create a new `file.api` file in the `api` directory with the following content.  

```
syntax = "v1"

import "download.api"
import "upload.api"
```

Enter fullscreen mode Exit fullscreen mode

This way we import both `Download` and `Upload` services like `#include` in `C/C++`. But there are a few things to keep in mind.

1.  the defined structs cannot be renamed
2.  the `service name` must be the same in all files

> The outermost `API` file can also contain part of the same `service` definition, but we recommend to keep it symmetrical, unless these `API`s really belong to the parent level, e.g. the same logical level as `Download` and `Upload`, then they should not be defined in `file.api`.

At this point, we have the following file structure.  

```
.
└── api
    ├── download.api
    ├── file.api
    └── upload.api
```

Enter fullscreen mode Exit fullscreen mode

### [](https://dev.to/kevwan/best-practices-on-developing-monolithic-services-in-go-3c95#generating-monolithic-service)Generating monolithic service

Now that we have the `API` interface defined, the next step is pretty straightforward for `go-zero` (of course, defining the `API` is pretty straightforward, isn't it?). Let's use `goctl` to generate the monolithic service code.  

```
$ goctl api go -api api/file.api -dir .
```

Enter fullscreen mode Exit fullscreen mode

Let's take a look at the generated file structure.  

```
.
├── api
│ ├── download.api
│ ├── file.api
│ └── upload.api
├── etc
│ └── file-api.yaml
├── file.go
├─ go.mod
├── go.sum
└── internal
    ├─ config
    │ └─ config.go
    ├─ handler
    │ ├── downloadhandler.go
    │ ├── routes.go
    │ └── uploadhandler.go
    ├─ logic
    │ ├── downloadlogic.go
    │ └── uploadlogic.go
    ├── svc
    │ └── servicecontext.go
    └─ types
        └─ types.go
```

Enter fullscreen mode Exit fullscreen mode

Let's explain the layout of the project by directory.

-   `api` directory: the `API` interface description file we defined earlier, no need to talk much
-   `etc` directory: this is for the `yaml` configuration files, all configuration items can be written in the `file-api.yaml` file
-   `file.go`: the file where the `main` function is located, with the same name as `service`, removed `-api` suffix
-   `internal/config` directory: the configuration definition of the service
-   `internal/handler` directory: the `handler` implementation of the routes defined in the `API` file
-   `internal/logic` directory: used to put the business processing logic corresponding to each route, the reason for the distinction between `handler` and `logic` is to minimize the dependency of the business processing part, to isolate `HTTP requests` from the logic processing code, and to facilitate the subsequent splitting into `RPC service` as needed
-   `internal/svc` directory: used to define the dependencies for business logic processing, we can create the dependent resources in `main` and pass them to `handler` and `logic` via `ServiceContext`
-   `internal/types` directory: defines the `API` request and response data structure

Let's not change anything, let's run it and see how it works.  

```
$ go run file.go -f etc/file-api.yaml
Starting server at 0.0.0.0:8888...
```

Enter fullscreen mode Exit fullscreen mode

### [](https://dev.to/kevwan/best-practices-on-developing-monolithic-services-in-go-3c95#implementing-the-business-logic)Implementing the business logic

Next we need to implement the relevant business logic, but the logic here is really just for demonstration purposes, so don't pay too much attention to the implementation details, just understand that we should write the business logic in the `logic` layer.

The following things are done here.

-   Add the `Path` setting in the configuration item to place the uploaded files, and by default I wrote the current directory, because it is an example, as follows.

```
type Config struct {
  RestConf
  // New
  Path string `json:",default=." `
}
```

Enter fullscreen mode Exit fullscreen mode

-   Adjusted the request body size limit as follows.

```
Name: file-api
Host: localhost
Port: 8888
# New
MaxBytes: 1073741824
```

Enter fullscreen mode Exit fullscreen mode

-   Since `Download` needs to write the file to the client, we passed `ResponseWriter` as `io.Writer` to the `logic` layer, and the modified code is as follows

```
func (l *DownloadLogic) Download(req *types.DownloadRequest) error {
  logx.Infof("download %s", req.File)
  body, err := ioutil.ReadFile(req.File)
  if err ! = nil {
    return err
  }

  n, err := l.writer.Write(body)
  if err ! = nil {
    return err
  }

  if n < len(body) {
    return io.ErrClosedPipe
  }

  return nil
}
```

Enter fullscreen mode Exit fullscreen mode

-   Since `Upload` needs to read the files uploaded by the user, we pass `http.Request` to the `logic` layer and the modified code is as follows.

```
func (l *UploadLogic) Upload() (resp *types.UploadResponse, err error) {
  l.r.ParseMultipartForm(maxFileSize)
  file, handler, err := l.r.FormFile("myFile")
  if err ! = nil {
    return nil, err
  }
  defer file.Close()

  logx.Infof("upload file: %+v, file size: %d, MIME header: %+v",
    handler.Filename, handler.Size, handler.Header)

  tempFile, err := os.Create(path.Join(l.svcCtx.Config.Path, handler.Filename))
  if err ! = nil {
    return nil, err
  }
  defer tempFile.Close()
  io.Copy(tempFile, file)

  return &types.UploadResponse{
    Code: 0,
  }, nil
}
```

Enter fullscreen mode Exit fullscreen mode

Full code: [https://github.com/zeromicro/zero-examples/tree/main/monolithic](https://github.com/zeromicro/zero-examples/tree/main/%20monolithic)

We can start the `file` monolithic service by running the following command.  

```
$ go run file.go -f etc/file-api.yaml
```

Enter fullscreen mode Exit fullscreen mode

The `Download` service can be verified with `curl`:  

```
$ curl -i "http://localhost:8888/static/file.go"
HTTP/1.1 200 OK
Traceparent: 00-831431c47d162b4decfb6b30fb232556-dd3b383feb1f13a9-00
Date: Mon, 25 Apr 2022 01:50:58 GMT
Content-Length: 584
Content-Type: text/plain; charset=utf-8

...
```

Enter fullscreen mode Exit fullscreen mode

The sample repository contains `upload.html`, the browser can open this file to try the `Upload` service.

## [](https://dev.to/kevwan/best-practices-on-developing-monolithic-services-in-go-3c95#summary-of-monolithic-development)Summary of monolithic development

Let me summarize the complete process of developing a monolithic service with `go-zero` as follows.

1.  define the `API` files for each submodule, e.g. `download.api` and `upload.api`
2.  define the general `API` file, e.g. `file.api`. Use it to `import` the `API` files of each submodule defined in step 1
3.  generate the monolithic service framework code via the `goctl api go` command
4.  add and adjust the configuration to implement the business logic of the corresponding submodule

In addition, `goctl` can generate `CRUD` and `cache` code according to `SQL` in one shot, which can help you develop monolithic services more quickly.

## [](https://dev.to/kevwan/best-practices-on-developing-monolithic-services-in-go-3c95#project-address)Project address

[https://github.com/zeromicro/go-zero](https://github.com/zeromicro/go-zero)

Welcome to use `go-zero` and **star** to support us!