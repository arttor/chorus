Project overview is in README.md it also describes project structure and components, it has links to components README and documentation website.

# build and test
We use Makefile to buld test and lint project.

But you can also use `go test ./...` to run all tests or for particular package. All test does not require any setup or cleanup even e2e, all tests can be run with `go test`.
Makefile also contains examples how to run formatters and linters and generate grpc servers and clients from protofiles, all required tools are installed with go 1.24 `go tool` feature.

Project should be able to compile and tests should always pass. 

## Flaky tests
Follwong packages contain flaky tests which can fail occasionally. Simple restart shoudl fix them:
- pkg/rpc/proxy_redis_request_reply_test.go
- any test in test/ and its nested directories
Here is example of flaky test output form github action logs:
```
    proxy_redis_request_reply_test.go:37: 
        	Error Trace:	/home/runner/work/chorus/chorus/pkg/rpc/proxy_redis_request_reply_test.go:37
        	Error:      	Received unexpected error:
        	            	NotImplemented: proxy is not listening
        	Test:       	TestProxyClient_GetCredentials

--- FAIL: TestApi_Bucket_CRUD (5.01s)
    bucket_test.go:147: 
        	Error Trace:	/home/runner/work/chorus/chorus/test/bucket_test.go:147
        	Error:      	Condition never satisfied
        	Test:       	TestApi_Bucket_CRUD

--- FAIL: TestApi_Bucket_List (3.01s)
    bucket_test.go:220: 
        	Error Trace:	/home/runner/work/chorus/chorus/test/bucket_test.go:220
        	Error:      	Condition never satisfied
        	Test:       	TestApi_Bucket_List

--- FAIL: TestApi_Object_Multipart (3.01s)
    multipart_test.go:28: 
        	Error Trace:	/home/runner/work/chorus/chorus/test/multipart_test.go:28
        	Error:      	Condition never satisfied
        	Test:       	TestApi_Object_Multipart
--- FAIL: TestApi_Object_CRUD (3.01s)
    object_test.go:44: 
        	Error Trace:	/home/runner/work/chorus/chorus/test/object_test.go:44
        	Error:      	Condition never satisfied
        	Test:       	TestApi_Object_CRUD
--- FAIL: TestApi_Object_Folder (3.01s)
    object_test.go:221: 
        	Error Trace:	/home/runner/work/chorus/chorus/test/object_test.go:221
        	Error:      	Condition never satisfied
        	Test:       	TestApi_Object_Folder
--- FAIL: TestApi_ZeroDowntimeSwitch (18.02s)
    switch_test.go:175: repl events 4 4
    switch_test.go:184: start write 2025-06-05 11:55:07.65899613 +0000 UTC m=+42.186492407
    switch_test.go:282: switch created 2025-06-05 11:55:08.327831208 +0000 UTC m=+42.855327475
    switch_test.go:255: 
        	Error Trace:	/home/runner/work/chorus/chorus/test/migration/switch_test.go:255
        	            				/opt/hostedtoolcache/go/1.24.3/x64/src/runtime/asm_amd64.s:1700
        	Error:      	Received unexpected error:
        	            	NoSuchUpload: NoSuchUpload
        	            		status code: 404, request id: , host id: 
        	Test:       	TestApi_ZeroDowntimeSwitch
    switch_test.go:306: 
        	Error Trace:	/home/runner/work/chorus/chorus/test/migration/switch_test.go:306
        	Error:      	Condition never satisfied
        	Test:       	TestApi_ZeroDowntimeSwitch

--- FAIL: TestApi_Object_Multipart (3.01s)
    multipart_test.go:28: 
        	Error Trace:	/home/runner/work/chorus/chorus/test/multipart_test.go:28
        	Error:      	Condition never satisfied
        	Test:       	TestApi_Object_Multipart
--- FAIL: TestApi_Object_CRUD (3.01s)
    object_test.go:44: 
        	Error Trace:	/home/runner/work/chorus/chorus/test/object_test.go:44
        	Error:      	Condition never satisfied
        	Test:       	TestApi_Object_CRUD
--- FAIL: TestApi_Object_Folder (3.01s)
    object_test.go:221: 
        	Error Trace:	/home/runner/work/chorus/chorus/test/object_test.go:221
        	Error:      	Condition never satisfied
        	Test:       	TestApi_Object_Folder
--- FAIL: TestApi_Bucket_List (3.01s)
    bucket_test.go:220: 
        	Error Trace:	/home/runner/work/chorus/chorus/test/bucket_test.go:220
        	Error:      	Condition never satisfied
        	Test:       	TestApi_Bucket_List
--- FAIL: TestApi_Bucket_CRUD (5.01s)
    bucket_test.go:147: 
        	Error Trace:	/home/runner/work/chorus/chorus/test/bucket_test.go:147
        	Error:      	Condition never satisfied
        	Test:       	TestApi_Bucket_CRUD
```
## Project structure

Here is projects structure with brief description of directories and packages
```
├── cmd                       #standard go entrypoint with main files 
│   ├── agent
│   ├── chorus
│   ├── proxy
│   └── worker
├── deploy                    # helm chart
├── docker-compose            # example of docker-compose local playground
├── docs                      # design documents
├── pkg
│   ├── api                   # implementation of grpc server for management api
│   ├── config                # common config logic
│   ├── ctx                   # utils for go context
│   ├── dom                   # common domain objects like errors and global types
│   ├── features              # feature flag logic
│   ├── lock                  # distributed lock serivice based on redis
│   ├── log                   # common logging 
│   ├── meta                  # serivice storing chorus metadata in redis
│   ├── metrics
│   ├── notifications         # s3 bucket notifications common logic
│   ├── policy                # responsible for routing and replication policies
│   ├── ratelimit             # ratelimit based on redis
│   ├── rclone                # rclone wrapper to copy object
│   ├── replication           # common logic to create replication tasks from events captured by proxy or agent
│   ├── rpc                   # allows communication between components based on redis pubsub
│   ├── s3                    # s3 common logic and constants
│   ├── s3client              # s3 client
│   ├── storage               # kv store based on redis
│   ├── tasks                 # common logic for asynq tasks
│   ├── trace
│   └── util
├── proto                     # contains protofiles for management api along with genrated files and plugin configs for grpc gateway
├── service                   # chorus compmonent (service)-specific logic
│   ├── agent                 # chorus agent
│   ├── proxy                 # chorus proxy
│   │   └── router
│   ├── standalone            # builds chorus standalone version 
│   └── worker                # chorus worker
│       ├── handler           # all asynq task handlers located in this package
├── test                      # e2e tests - run chorus worker, proxy, redis and im-mem s3 storages from go
│   ├── agent                 # e2e tests for chorus agent setup
│   └── migration             # e2e tests for s3 migration 
├── tools                     # tools for chorus - separate go modules
│   ├── bench                 # chorus benchmark tool - not relevant and not maintained - ignore
│   └── chorctl               # CLI client for chorus rgpc management api
└── ui                        # web ui client for chrous management api. written in vue framework and typescript. ignore this directory unless task explicitly asks to work on web ui
```

