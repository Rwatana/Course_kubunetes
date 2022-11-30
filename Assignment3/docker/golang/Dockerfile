# Specify the base image for the go app.
FROM --platform=linux/arm64 golang:1.19

WORKDIR /go/src/github.com/postgres-go
# Copy everything from this project into the filesystem of the container.
COPY . .
# Obtain the package needed to run code. Alternatively use GO Modules. 
RUN go mod download && go mod verify 

WORKDIR /usr/ryoma/go
# Add the contents of the host OS . /go contents to the working directory
ADD ./go .