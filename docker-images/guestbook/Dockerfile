
FROM golang:1.11.1 as builder

# Install required system dependencies
RUN apt-get install -y git

# Set up GOPATH and disable modules
ENV GOPATH /go
RUN mkdir -p /go/src/app
WORKDIR /go/src/app

# Install specific dependency versions using Git checkout
RUN git clone https://github.com/codegangsta/negroni.git $GOPATH/src/github.com/codegangsta/negroni && \
    cd $GOPATH/src/github.com/codegangsta/negroni && \
    git checkout v0.3.0

RUN git clone https://github.com/gorilla/mux.git $GOPATH/src/github.com/gorilla/mux && \
    cd $GOPATH/src/github.com/gorilla/mux && \
    git checkout v1.7.3  # Last version before Go 1.18+ features

RUN git clone https://gitlab.com/utd-admin/simpleredis.git $GOPATH/src/gitlab.com/utd-admin/simpleredis && \
    cd $GOPATH/src/gitlab.com/utd-admin/simpleredis && \
    git checkout main

# Copy and build application
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
