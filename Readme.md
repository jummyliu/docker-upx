# UPX

## Overview
A small image for usage in multi-stage Docker builds to compress binary files like Go or Rust. Based on the (scratch | official busybox image) and build via multi-stage build himself to make the image. For more information on the great tool UPX check out their [GitHub project](https://github.com/upx/upx)!

## How to build

```shell
docker build -t jummyliu/upx:scratch -f Dockerfile.scratch .

docker build -t jummyliu/upx:busybox -f Dockerfile.busybox .
```

## Usage

### CMD
To compress any file run following command.

```shell
docker run --rm -w `pwd` -v `pwd`:`pwd` jummyliu/upx --best --lzma {COMPRESSED_FILE_NAME} {FILE_NAME}
```

### Docker multi-stage build
For this example I used the official example of the Docker documentation for multi-stage builds

```Dockerfile
FROM golang:alpine as builder
COPY ./ /www/
WORKDIR /www
RUN GO111MODULE=on GOARCH=amd64 GOOS=linux CGO_ENABLED=0 go build -mod vendor -ldflags "-s -w" -trimpath -o app

FROM jummyliu/upx as upx
COPY --from=builder /www/app /www/app.org
RUN ["upx", "--best", "--lzma", "-o", "/www/app", "/www/app.org"]

FROM scratch
COPY --from=upx /www/app /www/
WORKDIR /www
CMD ["./app"]
```
