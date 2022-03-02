# Remote Job Worker Service



**Table of Contents**

[TOCM]

[TOC]

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
		
- Out of scope: Can admin users view jobs created by other admin users? Who will guard the guards- This interaction is ignored for the prototype.

## User Actions

| Action  | Outcome  | Supported by User Type |
| :------------ |:---------------:| -----:|
| Connect to service      | AuthN service and client(mTLS). AuthZ user and privilege  | Same behavior for Admin and standard user |
| Start Job      | Accept the user job to run on machine. Return Job ID, error code to denote success(nil error) <br/> or failure of starting the job(start failure reason) | Same behavior for Admin and standard user |
| Stop Job      |  Accepts a Job ID to stop. Returns error code to denote success(nil error)<br/> or failure of stopping the job(stop failure reason)       |   Admin user can stop job started by any user. |
| Query Status | In progress or completed()       |    $1 |
| Stream output | are neat        |    $1 |

## Out of Scope

- User creation and management: We will use pre defined 1 standard user and 1 admin user


## High Level Design


## Low Level Design

