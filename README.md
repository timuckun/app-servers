## Table of Contents

* [Scope](#scope)
  * [Hello World](#hello-world)
  * [Platform](#platform)
  * [Wrk](#wrk)
* [Languages](#languages)
  * [Ruby](#ruby)
  * [Elixir](#elixir)
  * [Node.js](#nodejs)
  * [GO](#go)
  * [Java](#java)
  * [Nim](#java)
  * [Crystal](#crystal)
* [Benchmarks](#benchmarks)
  * [Results](#results)
  * [Rack](#rack)
  * [Plug](#plug)
  * [Node Cluster](#node-cluster)
  * [ServeMux](#servemux)
  * [Jetty](#jetty)
  * [asynchttpserver](#asynchttpserver)
  * [Crystal HTTP](#crystal-http)

## Scope
The idea behind this repository is to test how HTTP libraries for different languages behave under heavy loading.   

### Hello World
The "application" i tested is barely minimal: it is the HTTP version of the "Hello World" example.

### Platform
I registered these benchmarks with a MacBook PRO 15 late 2011 having these specs:
* OSX El Captain
* 2,2 GHz Intel Core i7 (4 cores)
* 8 GB 1333 MHz DDR3

### Wrk
I used [wrk](https://github.com/wg/wrk) as the loading tool.
I measured each application server three times, picking the best lap.  
Here is the common script i used:

```
wrk -t 4 -c 50 -d30s --timeout 2000 http://127.0.0.1:<port>
```

## Languages
I chose to test the following languages/runtime:

### Ruby
[Ruby](https://www.ruby-lang.org/en/) 2.3 is installed via homebrew.  
Ruby is the language i have more experience with.  
I find it an enjoyable language to code with, with a plethora of good libraries and a lovely community.

### Elixir
[Elixir](http://elixir-lang.org/) 1.3 is installed via homebrew.  
I studied Elixir in 2015, surfing the wave of [Prag-Dave](https://pragdave.me/) enthusiasm and finding its *rubyesque* resemblance inviting.  
Being based on [Erlang](https://www.erlang.org/) it supports parallelism out of the box by spawning small (2Kb) processes.

### Node.js
[Node.js](https://nodejs.org/en/) stable version (4.x) is installed by official OSX package.  
I once used to code in JavaScript much more than today. I left it behind in favor or more "backend" languages: it is a shame, since V8 is pretty fast, ES6 has filled many language lacks and the rise of Node.js has proven JavaScript is much more than an in-browser tool.

### GO
[GO](https://golang.org/) language version 1.6.2 is installed by official OSX package.  
GO focuses on simplicity by intentionally lacking features considered redundant (an approach i am a fan of). It tries to address verbosity by using type inference, duck typing and a dry syntax.  
At the same time GO takes a straight approach to parallelism, coming with built in [CSP](https://en.wikipedia.org/wiki/Communicating_sequential_processes) and green threads (goroutines).  

### Java
[Java](https://www.java.com/en/) 8 comes pre-installed on Xcode 7.31.  
I get two Sun certifications back in 2006 and realized the more i delved into Java the less i liked it.
Ignoring Java on this comparison is not an option anyway: Java is the most used programming language in the world (2016) and some smart folks have invested on it since the 90ies.

### Nim
[Nim](http://nim-lang.org/) 0.14.2 is installed from source.  
Nim is an efficient, elegant, expressive strong typed, compiled language.
Nim supports metaprogramming, functional, message passing, procedural, and object-oriented coding style. It also provides several compiling options to better adapt to the running environment.

### Crystal
[Crystal](http://crystal-lang.org/) 0.18.4 is installed via homebrew.  
Crystal has a syntax very close to Ruby, but brings some desirable features such as strong typing (hidden by a pretty smart type inference algorithm) and ahead of time compilation.  
For concurrency Crystal adopts the CSP model (like GO) and evented/IO to avoid blocking calls, but parallelism is not yet supported.

## Benchmarks
I decided to test each language by using the standard/built-in HTTP library.

### Results
Here are the benchmarks results ordered by increasing throughput.

| App Server                             | Throughput (req/s) | Latency in ms (avg/stdev/max) |
| :------------------------------------- | -----------------: | ----------------------------: |
| [Rack](#rack)                          |          29348.90  |              1.62/0.33/29.91  |
| [Plug](#plug)                          |          29878.35  |              2.69/4.54/83.49  |
| [Node Cluster](#node-cluster)          |          47053.76  |              1.44/2.31/88.66  |
| [asynchttpserver](#asynchttpserver)    |          47351.25  |              1.01/0.15/10.93  |
| [Jetty](#jetty)                        |          52009.95  |               0.91/0.14/9.87  |
| [ServeMux](#servemux)                  |          57135.76  |               0.82/0.17/4.98  |
| [Crystal HTTP](#crystal-http)          |          69467.38  |               0.68/0.20/4.09  |

### Rack
I tested ruby by using a plain [Rack](http://rack.github.io/) application with the [Puma](#http://puma.io/) application server.  

##### Bootstrap
```
bundle exec puma -w 8 --preload -t 16:32 app.ru
```

##### Considerations
Rack proves to be a pretty fast HTTP server (at least among scripting languages): it's modular, easy to extend and almost every Ruby Web framework is Rack-compliant.
The ability to add middlewares easily makes Rack so flexible to make it my first choice in place of heavyweight frameworks (that can be added in a second time).  

### Plug
I tested Elixir by using [Plug](https://github.com/elixir-lang/plug) library that provides a [Cowboy](https://github.com/ninenines/cowboy) adapter.

##### Bootstrap
```
iex -S mix
iex> c "lib/plug_server.ex"
iex> {:ok, _} = Plug.Adapters.Cowboy.http PlugServer, [], port: 9292
```

##### Considerations
Elixir performance are pretty solid but not stellar.  
To be fair Erlang, and by reflection Elixir, benefits most by using the [OTP library](#http://elixir-lang.org/getting-started/mix-otp/supervisor-and-application.html) to grant reliability and resilience over a distributed system.  
Being an immutable language, Elixir optimizes memory usage when serving articulated views.

### Node Cluster
I used Node cluster library to spawn one process per CPU, thus granting parallelism (as with Ruby).

##### Bootstrap
```
node node_server.js
```

##### Considerations
While it is true that Node.js suffers JavaScript single threaded nature, it delivered very solid performance: Node's throughput is on par with slowest compiled languages (but consistency is worst).

### ServeMux
I opted for the [HTTP ServeMux](https://golang.org/pkg/net/http/) GO standard library.

##### Bootstrap
```
go build go_server.go
./go_server
```

##### Considerations
GO is a pretty fast language and allows using all of the cores with no particular configuration.  
The usage of small green threads allows GO to tolerate high loads of requests with very good latency.  

### Jetty
To test Java i used [Jetty](http://www.eclipse.org/jetty/): a modern, stable and quite fast servlet container.  

##### Bootstrap
```
javac -cp javax.servlet-3.0.0.v201112011016.jar:jetty-all-9.2.14.v20151106.jar HelloWorld.java
java -cp .:javax.servlet-3.0.0.v201112011016.jar:jetty-all-9.2.14.v20151106.jar -server HelloWorld
```

##### Considerations
I know Java is pretty fast nowadays: thousands of optimizations have been done to the JVM during the last two decades.  
Jetty uses one thread per connection, delivering consistent results (accepting JVM memory consumption and warm-up times) 

### asynchttpserver
I used the Nim asynchttpserver module to implement a high performance asynchronous server.  

##### Bootstrap
```
nim c -d:release --threads:on nim_server.nim
./nim_server
```

##### Considerations
Nim proved to keep its promises, being a fast and concise language.  
From a pure performance point of view Nim is faster than Ruby and Elixir, substantially on par with Node, slower than Java, GO and Crystal.  

### Crystal HTTP
I used Crystal HTTP server standard library.  
Crystal uses green threads, called "fibers", spawned inside an event loop via the [libevent](http://libevent.org/) library.

##### Bootstrap
```
crystal compile ./server/crystal_server.cr --release
./crystal_server
```

##### Considerations
Crystal language recorded the best lap of the pack, outperforming more mature languages like Java and GO, but also new-kids-on-the-block ones like Nim.  
This is even more interesting considering the language executes on a single thread only.
