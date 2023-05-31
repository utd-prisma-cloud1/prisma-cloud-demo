FROM alpine:3.17.3
LABEL purpose="utd-cnsp"
RUN apk update && apk add ruby ruby-dev libffi-dev gcc make g++
RUN gem install redis sinatra:3.0.5 thin hiredis --ignore-dependencies
COPY /app/app.rb /app/app.rb

RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

EXPOSE 8080
CMD ["ruby", "/app/app.rb"]
