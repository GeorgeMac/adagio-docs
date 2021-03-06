Adagio Overview
---------------

Adagio is designed to facilitate workflow execution across a fleet of agents via an API known as the control plane.

![adagio architecture diagram](/architecture.svg)

### Workflow

> What is a workflow?

{{< mermaid >}}
graph LR;
  a --> c;
  a --> d;
  b --> d;
  b --> f;
  c --> e;
  d --> e;
  e --> g;
  f --> g;

  a(("✓"));
  b(("✓"));
  c(("✓"));
  d(("✗"));
  e(("..."));
  f(("✓"));
  g(("..."));
  style a fill:#0f0, stroke:#000;
  style b fill:#0f0, stroke:#000;
  style c fill:#0f0, stroke:#000;
  style d fill:#f00, stroke:#000;
  style e fill:#fff, stroke:#000;
  style f fill:#0f0, stroke:#000;
  style g fill:#fff, stroke:#000;
{{< /mermaid >}}

A workflow is a set of instructions for work defined in a graph structure. In particular a directed acyclic-graph. Meaning there are no cycles between nodes (vertices).
This ensures a topological sort can be performed on the workflow in order to plan and execute the order of work.

The workflow graph is serialized and sent to the control plane to be executed. The definitions for the graph model are defined using protobuf and can be
found within the `pkg/adagio` Go package in this project.
Each node (vertex) in the graph can be thought of as a single function call. The function has a signature defined within the specification of a node along
with arguments encoded as metadata.

Alongside this; a node can be further defined with automatic recovery instructions in the form of a number of retries for a specific error or failure condition.
This allows for recovery to be performed automatically by the agents deployed.

### Node

> What is a node?


{{< mermaid >}}
graph LR;
  a(("f(args)"));
  b(("..."));
  a --> b;
  style a fill:#fff, stroke:#000;
  style b fill:#fff, stroke:#fff;
{{< /mermaid >}}

A node at its core is a specification of work. The node definition contains a type called the node spec as its first property.
This node specification contains the function to be called, the metadata associated (arguments) and any retry specifications.
Alongside this specification there are some runtime properties which record the result conclusion and output of each execution attempt,
the creation and finish times, along with any inputs fed in from dependent nodes.

### Agent

> What is an agent?

An agent is a single thread of execution which attempts to claim, execute and report on a single node at a time.
One process could spawn multiple agents e.g. in separate go routines. This can be achieved using the `pkg/agent.Pool` with
an agent count of two or more.

When a workflow is instantiated (started) it becomes known as a *run*. Workers are notified of the nodes within the workflow run.
In particular, they are notified of the ready nodes. Initially these are the nodes within the graph with no inbound edges.

{{< mermaid >}}
graph LR;
  a(("ready"));
  b(("ready"));
  c(("not ready"));
  a --> c;
  b --> c;
  style a fill:#fff, stroke:#000;
  style b fill:#fff, stroke:#000;
  style c fill:#fff, stroke:#000;
{{< /mermaid >}}

A node becomes ready once all nodes which feed into it are completed with a successful conclusion.
Agents consume nodes, not workflow runs. This allows for execution of a workflow to be distributed across multiple agents.

An agent will only claim nodes which it can execute. This is decided based on the nodes specification runtime property.
If the agent has a function definition associated for the runtime it can execute it. Given this is the case it attempts to make a *claim*.

Only one agent can successfully make a claim for a node. This is how we ensure nodes are executed by only one worker at a time.
Once the work has been performed the result is recorded, the graph state is updated in the database and any _newly ready_ nodes
are communicated with listening agents.

When an agent holds a claim on a node it must reported back a heartbeat while it performs execution.
A failure to do so will cause all agents to be notified of an orphaned node. An orphaned node will
be reclaimed by another agent and the result of execution be recorded as an error.
Given the node specifies a number of retries on the `error` condition the node may be moved back into the
ready state to be claimed again. This is how fail-over can be configured in the event an agent becomes "unhealthy".

> We recommend functions be idempotent in order to be retried safely.

Agents are built to facilitate your workflow needs and are the concern of operators and function providers.

### Control Plane API

> What is the control plane API?

The control plane is the entry point to get work done. It exposes a number of API actions to instantiate (start) workflows as runs, list existing runs
in the system, list available agents (for introspection purposes), inspect individual runs or just to get high level statistics on the overall counts
of runs, nodes and their states within the cluster.

It is exposed as a gRPC API, however, there is a pre-built json API gateway which can be deployed and communicated with the gRPC one. This API comes
with an accompanying swagger specification. This has been used to generate the javascript client used within the adagio UI.

The control plane API is designed to serve your cluster consumers who need to execute workflows.
It needs to be operated, but is intended to be simple to deploy and monitor.
