FROM golang:1.15.0-alpine3.12 AS builder

ENV GO111MODULE on
ENV GOPROXY https://goproxy.cn

RUN apk add git --no-cache \
        && go get -u -v github.com/shadowsocks/go-shadowsocks2

FROM alpine:3.12

WORKDIR /app
COPY --from=builder /go/bin/go-shadowsocks2 /app/
ENTRYPOINT ["./go-shadowsocks2"]