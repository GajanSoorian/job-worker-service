# Remote Job Worker Service



**Table of Contents**
- [Remote Job Worker Service](#remote-job-worker-service)
	- [Requirments](#requirments)
		- [Library](#library)
		- [API](#api)
		- [Client](#client)
	- [Users](#users)
	- [User Actions](#user-actions)
	- [High Level Design](#high-level-design)
	- [Out of Scope](#out-of-scope)
	- [Additional information](#additional-information)
		- [Security](#security)
		- [gRPC](#grpc)


## Requirments

### Library
+ Worker library with methods to start/stop/query status and get the output of a job.
+ Library should be able to stream the output of a running job.
	+ Output should be from start of process execution.
	+ Multiple concurrent clients should be supported.
+ Add resource control for CPU, Memory and Disk IO per job using cgroups.
### API
+ GRPC API to start/stop/get status/stream output of a running process.
+ Use mTLS authentication and verify client certificate. Set up strong set of cipher suites for TLS and good crypto setup for certificates. Do not use any other authentication protocols on top of mTLS.
+ Use a simple authorization scheme.
### Client
+ CLI should be able to connect to worker service and start, stop, get status, and stream output of a job.

## Users

1) Standard user: A standard user can start a job, with default/predefined CPU,Memory and I/O usage policy. Can only stop, request status and stream ouput for jobs they created.

2) Admin User: An Admin user can start jobs with elevated/customized CPU,Memory and I/O usage policy. Can stop, request status and stream ouput ANY jobs created by ANY user.
		
## User Actions

| Action  | Contract  | User Behavior |
| :------------ |:---------------| :-----|
| Start Job.      | Accept the user job to run on machine. Return Job ID, error code to denote success(nil error) <br/> or failure of starting the job(start failure reason). <br/> Associate the job id with user id to support "Query status" and "Stream output" actions. | Same behavior for Admin and standard user |
| Stop Job.      |  Accepts a Job ID to stop. Verify if job was created by the user(if standard user). Returns error code to denote success(nil error)<br/> or failure of stopping the job(stop failure reason).       |   Admin user can stop job started by any user. |
| Query Status. | Accepts a Job ID to stop. In progress or completed().       |    Admin user can query status of job started by any user. |
| Stream output | Check if a job is in progress.        |    Admin user can stream output of any job started by any user. |
| List jobs | Check if a job is in progress.        |    list all jobs triggered by user. Admin can list jobs triggered by other users. |

Prerequisite for every action: Authenticate service and client(mTLS). Authorization check for user, check user privilege with job request.

Each action will be implemented as a gRPC API.

## High Level Design

Architecture:

 ![Architecture of the system](Architecture.png)

Flow Diagram for Create and Stop job Actions:

 ![Flow Diagram for Create and Stop job](FlowDiagram-CreateStop.png)

Flow Diagram for Query Status and Stream Output Actions:

![Flow Diagram for Create and Stop job](FlowDiagram-QueryStatus-StreamOutput.png)

## Out of Scope

- User creation and management: We will use pre defined 1 standard user and 1 admin user.
- Job Registry component does not provide persistence. will store in-memory and use pre defined clean up rule.
- Worker Monitor:
  -  No support for registering new workers/monitoring health
  -  Will use simple round-robin to choose the worker machine.
  -  For this prototype we will assume we have only one worker service(so only one linux machine)
- Pre existing security tokens already exist for Job Runner service and Job Worker service.
- List jobs makes it easy for user and admin to perform other actions. Low Priority for now.
- Versioning support for APIs.

## Additional information

### Security

mTLS will be used to secure client and service request.

Signature scheme: Ed25519

Benefits: Faster than RSA, more secure than ECDSA

Request Authorization : Pre defined API keys can be used. (May revisit this in the coming days)

Action Authorization: Application logic that links job id with user id can be used to check priviledge for 

### gRPC

Service APIs will be gRPC endpoints.

Serialization/De-serialization library to be used with gRPC consideration - Flatbuffers vs google Protobuf:

I ran a small PoC to benchmark serializing and deserializing performance of Google Protobuf and Flatbuffers for a simple message(string and int) of size 8 kb. The flatbuffer lib performance was promising:
	Pros: Flatbuffers have an almost negligible de-serialization time compared to protobuf
	Cons: The serialized payload size was 9% larger than the payload serialized by protobuf
	Nice bonus: Flatbuffers schema compiler can generate .fbs schema from a proto file. 

Payload size, flexibility, learning curve for Flatbuffers will be evaluated and a switch will be made to protobuf if the trade off is not worth it.

Protobuf schema is define [here](../api/proto/jobService.proto). Flatbuffer schema is defined [here](../api/fbs/jobService.fbs).