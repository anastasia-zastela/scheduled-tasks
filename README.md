# TaskMaster: Go-Based Distributed Task Scheduler

TaskMaster is a robust and efficient task scheduler built for educational purposes using Go. It's designed to handle a high volume of tasks and distribute them across multiple workers for execution.

## System Components

TaskMaster comprises several essential components that collaborate to schedule and execute tasks. Here’s a brief overview of each:

- **Scheduler**: Acts as the front-end server, receiving tasks from clients and scheduling them for execution.

- **Coordinator**: Selects tasks that need execution based on their schedules, manages worker registration and decommissioning, and distributes tasks to available workers.

- **Worker**: Executes tasks assigned by the coordinator and reports the completion status back. Workers register with the coordinator and send heartbeats to indicate they are active.

- **Client**: Submits tasks to the scheduler via an HTTP endpoint and queries the status of their tasks.

- **Database**: A PostgreSQL database stores task details, scheduling information, and completion statuses. The scheduler and coordinator interact with the database to retrieve and update task information.

Each component is a separate service, communicating via gRPC, allowing high scalability and fault tolerance.

## Task Lifecycle

### Scheduling

1. Users schedule a task by sending an HTTP request to the scheduler.
2. The scheduler saves the task and its scheduling information in the database.

### Execution

1. The coordinator scans the database periodically to find scheduled tasks.
2. The coordinator distributes these tasks to workers.
3. Workers queue the tasks in memory and execute them as soon as resources are available.

### Retries

1. The coordinator retries task submission to workers if the initial attempt fails.
2. Tasks that fail during execution on a worker are not retried.

### Limitations

1. TaskMaster is an educational project and is not intended for production use.
2. Tasks are simple strings representing arbitrary data, focusing on demonstrating efficient task scheduling and scaling.
3. The coordinator’s push-based approach can overwhelm workers if insufficient workers are available.

## Directory Structure

An overview of the project’s directory structure:

- [`cmd/`](./cmd/): Entry points for the scheduler, coordinator, and worker services.
- [`pkg/`](./pkg/): Core logic for the scheduler, coordinator, and worker services.
- [`data/`](./data/): SQL scripts to initialize the database.
- [`tests/`](./tests/): Integration tests.
- [`*-dockerfile`](./docker-compose.yml): Dockerfiles for building the scheduler, coordinator, and worker services.
- [`docker-compose.yml`](./docker-compose.yml): Docker Compose configuration for starting the entire cluster.

## Launching the Cluster

To start a complete cluster using Docker Compose, run:

```sh
docker-compose up --build --scale worker=3
```

This command builds Docker images for the coordinator, scheduler, and worker services, then starts the cluster with one coordinator, one scheduler, and three workers. The `--scale` option specifies the number of workers.

Ensure you have Docker and Docker Compose installed to run this command.

## Interacting with the Cluster

Clients can interact with the TaskMaster cluster to schedule tasks and check their status via HTTP requests:

### Scheduling a Task

Send a POST request to `localhost:8081/schedule` with a JSON body including:

- `"command"`: The command to be executed.
- `"scheduled_at"`: The scheduled time in ISO 8601 format.

The response includes a `"task_id"` for querying the task’s status.

#### Example Schedule Request

```sh
curl -X POST localhost:8081/schedule -d '{"command":"<your-command>","scheduled_at":"2023-12-25T22:34:00+05:30"}'
```

### Checking Task Status

Send a GET request to `localhost:8081/status?task_id=<task-id>` where `<task-id>` is the ID returned when scheduling the task.

#### Example Status Request

```sh
curl localhost:8081/status?task_id=<task-id>
```