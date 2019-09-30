all: binary container clean

binary:
	go get -d -v ./...
	CGO_ENABLED=0 go build -o consumer .

container:
	docker build -t bwnetflow .

clean:
	rm ./consumer
