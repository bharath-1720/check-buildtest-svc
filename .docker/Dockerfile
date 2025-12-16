ARG GO_VERSION=1.23.4

FROM golang:${GO_VERSION}-alpine AS builder

# Install SSH and Git using apk
RUN apk add --no-cache git openssh

# Copy SSH key
COPY /id_rsa /root/.ssh/
RUN chmod 400 /root/.ssh/id_rsa
RUN ssh-keyscan github.com > /root/.ssh/known_hosts
RUN git config --global --add url."git@github.com:".insteadOf "https://github.com/"

# Set working directory
WORKDIR /app

# Copy module files and download dependencies
COPY go.mod .
COPY go.sum .
RUN go mod download

# Copy rest of the code and build
COPY . .
RUN go build -o ./server ./cmd/main.go

# Final stage
FROM golang:${GO_VERSION}-alpine AS app

# Install required system packages using apk
RUN apk add --no-cache ca-certificates ffmpeg

# Set working directory
WORKDIR /app

# Copy built binary and env file
COPY --from=builder /app/server .
COPY ./.env.prod /app/.env.prod
RUN cp /app/.env.prod /app/.env
# Expose port
EXPOSE 8081

# Run the server
CMD ["./server"] 
