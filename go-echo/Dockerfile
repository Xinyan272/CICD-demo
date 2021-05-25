FROM golang:latest

WORKDIR /go/src/app
COPY . .

# get echo
RUN go get -u github.com/labstack/echo
RUN go build .
EXPOSE 8080
CMD  ["./main"]
