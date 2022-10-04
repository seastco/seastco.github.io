---
title: Building Redis
date: 2022-10-02 22:44:00
keywords:
  - Go
  - Redis
  - CodeCrafters
---

I have a great deal of respect for anyone who’s built something novel. Whether it’s a programming language, database, or 
web framework, it’s such an amazing feat. I’m five years into my career and I personally wouldn’t know where to start 
with any of these. I wish there was a clear roadmap that outlined "do this and you’ll be good enough to build something 
like Redis." I think the roadmap is to just keep coding? That’s until I stumbled across [CodeCrafters](https://codecrafters.io), 
a Y Combinator-backed startup trying to teach engineers how to write complex software.

*Practice Writing Complex Software. Recreate Redis, Git, Docker - with your own hands. Gain expert-level confidence by 
taking action and diving deep, learning from the world’s best.* Exactly what I’ve been looking for. I signed up and 
dived immediately into the Redis tutorial.

Each stage in the tutorial has an explanation, solution, and sometimes an official source code walkthrough. I’ve 
provided a summary of each stage below, annotated with some of my thoughts. 

## 1. Bind to a port
First, start a TCP server on port 6379, the default port that Redis uses. This does not compile (yet) because listener 
is an unused variable (+1 for cool Go features).
```
func main() {
	listener, err := net.Listen("tcp", "0.0.0.0:6379")
	if err != nil {
		fmt.Println("Failed to bind to port 6379")
		os.Exit(1)
	}
}
```

## 2. Respond to PING
Next, respond to the [PING](https://redis.io/commands/ping/) command. Per the tutorial, hardcode the response and assume 
the client is sending "PING".

Redis clients & servers speak using RESP (REdis Serialization Protocol), so we need to encode the "PONG" response as a 
[RESP Simple String](https://redis.io/docs/reference/protocol-spec/). For RESP Simple Strings, the first byte of the 
response is "+" and different parts of the protocol are always terminated using "\r\n" (CLRF).
```
func main() {
	listener, err := net.Listen("tcp", "0.0.0.0:6379")
	if err != nil {
		fmt.Println("Failed to bind to port 6379")
		os.Exit(1)
	}
	
	conn, err := listener.Accept()
	if err != nil {
		fmt.Println("Error accepting connection: ", err.Error())
		os.Exit(1)
	}
	
	defer conn.Close()
	conn.Write([]byte("+PONG\r\n"))
}
```

## 3. Respond to multiple PINGs
The solution suggested to simply add a for loop to handle multiple client requests. Technically true, but the 
conn.Read() below is non-blocking and will not wait on client input. So as long as the connection is open, server is 
spamming the client with PONG regardless of what client is sending. I raised with the CodeCrafters team that this 
solution is misleading and they agreed to modify it in a future release.
```
for {
    if _, err := conn.Read([]byte{}); err != nil {
        fmt.Println("Error reading from client: ", err.Error())
        os.Exit(1)
    }
    
    conn.Write([]byte("+PONG\r\n"))
}
```

## 4. Handle concurrent clients
To handle multiple concurrent clients, we wrap listener.Accept() in a for loop and handle each connection using a 
*goroutine*. A *goroutine* is a lightweight thread of execution that executes concurrently with the calling one.
```
func main() {
	listener, err := net.Listen("tcp", "0.0.0.0:6379")
	if err != nil {
		fmt.Println("Failed to bind to port 6379")
		os.Exit(1)
	}
	
	for {
	    conn, err := listener.Accept()
	    if err != nil {
		    fmt.Println("Error accepting connection: ", err.Error())
		    os.Exit(1)
		}
		
		go handleConnection(conn)
	}
}

func handleConnection(conn net.Conn) {
    defer conn.Close()
    
    for {
        if _, err := conn.Read([]byte{}); err != nil {
            fmt.Println("Error reading from client: ", err.Error())
            continue
        }
        
        conn.Write([]byte(+PONG\r\n"))
    }
}
```

## 5. Implement ECHO, GET, & SET commands
Now that we’re dealing with multiple commands, we have to write a RESP parser to actually understand what clients are
sending our server. In short, this required implementing the [RESP protocol spec](https://redis.io/docs/reference/protocol-spec/). 
Once we’ve parsed out the command and respective args, the business logic is straightforward. A simple Map struct was 
used for GET/SET, and ECHO is just responding with the same content provided in the request. I spent a while trying to 
figure out how to condense this into this blog post, but it’s a loooong spec. The code is available on my 
[GitHub](https://github.com/seastco/redis-go/tree/main/app).

## Closing Thoughts
While this was an enjoyable exercise, I don’t think CodeCrafters is ready for primetime. I’d love to see more depth; 
I didn’t learn much about Redis other than the esoteric RESP protocol. The source code walkthroughs were valuable, but 
those were only provided for 2 of the 6 stages. I’ll revisit this in a year when the content has hopefully matured. The 
future roadmap must be good if they’re backed by Y Combinator. This kind of educational content is lacking and is absolutely 
needed, so I’m rooting for their success.

A final note - it’s interesting to see the Redis source code have [files](https://github.com/redis/redis/blob/7.0/src/server.c) 
with 7000+ LOC. If I blindly came across this code not knowing it was Redis, I’d have a smug, knee-jerk reaction about 
how the code is trash. A humble reminder to myself to not be so opinionated about subjective rules. What the hell do I 
know? Redis is one of the most loved and successful databases to date; clearly the maintainers know what they’re doing. 
I’m not saying 7000+ LOC is the correct thing to do, but there are obviously bigger fish to fry.

Until next time.