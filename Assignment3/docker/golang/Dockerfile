# Specify the base image for the go app.
FROM golang:1.19

WORKDIR /usr/ryoma/go
COPY . .

WORKDIR /usr/ryoma/go
RUN go mod tidy

# Add the contents of the host OS . /go contents to the working directory
ADD ./go .
