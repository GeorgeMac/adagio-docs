---
title: Adagio
type: docs
---

## Adagio a Workflow Engine

![Adagio Demo Gif](/adagio.gif)

{{< columns >}}

## Distributed

Adagio is built to be distributed. At its heart adagio is a library that enforces a protocol of collaboration via some backing datastore. This allows for multiple instances of agents to scale horizontally whilst consuming work.

<--->

## Expressive

Work is created in the system as a workflow. A workflow is a directed acyclic graph which describes the dependencies between the discrete pieces of work (the nodes). A piece of work is expressed as a single function call.

<--->

## Fault Tolerant

Things fail and Adagio expects them to. When a node execution becomes _unhealthy_; other agents react accordingly. When workflow authors configure nodes to be retried agents will reschedule new attempts automatically.

{{< /columns >}}
