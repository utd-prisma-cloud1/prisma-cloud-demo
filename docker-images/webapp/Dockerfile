FROM alpine:3.18
LABEL purpose="utd-cnsp"

RUN apk update && apk add ruby=3.2.4-r0 ruby-dev=3.2.4-r0 libffi-dev=3.4.4-r2 gcc=12.2.1_git20220924-r10 make=4.4.1-r1 g++=12.2.1_git20220924-r10 openssl
RUN gem install redis:5.2.0 sinatra:3.0.5 thin:1.8.2 hiredis:0.6.3 --ignore-dependencies
COPY /app/app.rb /app/app.rb

RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

EXPOSE 8080
CMD ["ruby", "/app/app.rb"]
