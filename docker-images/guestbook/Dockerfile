FROM golang as builder
RUN go env -w GO111MODULE=off
RUN go get github.com/codegangsta/negroni
RUN go get github.com/gorilla/mux gitlab.com/utd-admin/simpleredis
COPY main.go .
RUN go build main.go

#FROM busybox:ubuntu-14.04
FROM busybox:glibc
LABEL purpose="utd-cnsp"
COPY --from=builder /go//main /app/guestbook

ADD public/index.html /app/public/index.html
ADD public/script.js /app/public/script.js
ADD public/style.css /app/public/style.css
ADD public/jquery.min.js /app/public/jquery.min.js

RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

WORKDIR /app
CMD ["./guestbook"]
EXPOSE 3000
