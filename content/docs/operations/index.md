Adagio Operations
-----------------

### Deployment

```
     +---------------------------------------------------------------------+
     |                                                                     |
     |                                           scale horizontally        |
     |                                                                     |
     |  +-----------+  +-----------+  +-----------+               +-----+  |
     |  |           |  |           |  |           |               |     |  |
     |  |   agent   |  |   agent   |  |   agent   |               |  +  |  |
     |  |           |  |           |  |           |      ...      |     |  |
     |  +-----------+  +-----------+  +-----------+               +-----+  |
     |        |              |              |                              |
     +---------------------------------------------------------------------+
              |              |              |              |
              +--------------------------^-----------------------+ ...
                                         |            |
                                         |            |
                                         |            |
              +-----------------+ +-------------------v-------+    
              |                 | |                           |    
              |  ✓ etcd         | |                           |    
              |  - dynamoDB     | |  Multi-Row Transactional  |    
              |  - PostgresSQL  | |          Database         |    
              |  ...            | |                           |    
              |                 | |                           |    
              +-----------------+ +------^--------------------+    
                                         |            |
                                         |            |
              +---------------------------------------v----------+ ...
              |              |              |              |
     +---------------------------------------------------------------------+
     |        |              +              |                              |
     |        |        control plane        |    scale horizontally        |
     |        |              +              |                              |
     |  +-----------+  +-----------+  +-----------+               +-----+  |
     |  |           |  |           |  |           |               |     |  |
     |  |    api    |  |    api    |  |    api    |               |  +  |  |
     |  |           |  |           |  |           |      ...      |     |  |
     |  +-----------+  +-----------+  +-----------+               +-----+  |
     |                                                                     |
     +---------------------------------------------------------------------+
```

Adagio is designed to facilitate the scale of agents which perform work and the control plane API processes horizontally.
It doesn't attempt to dictate how or where you deploy your agents or the API tier. Rather it implements the pieces which
collaborate via some multi-row transactional database to ensure horizontal scale can be achieved.

As of today `adagiod` serves as both the control plane API and _an implementation_ of an adagio agent. This implementation has some
limited functionality which is expected to not be of much use to the adagio users. While work progresses on the pre-baked agent
within `adagiod` it should be made clear that it is a pre-configured binary wrapper for the `pkg/adagio` Go package. This package
is intended for consumers to import and build agents of their own in Go. Please see `pkg/adagio/doc.go` for more details on using
this package to bake your own adagio agent in Go. In the future the goal will be to enable other agent function implementations
in other languages. This may be through some IPC or network based API for example.
