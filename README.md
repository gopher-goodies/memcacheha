# memcacheha

memcacheha wraps `github.com/bradfitz/gomemcache/memcache` to provide HA (highly available) functionality with lazy client-side synchronization.

## Operation

### Failover condition assumptions

* Only one node will be lost at once
* A node (re)joining the cluster will be empty.

### Unconditional Writing

* Items will be concurrently written to all healthy nodes. The write will not return until:
	* All nodes have been written to and responded, or timed out

### Conditional Writing

* Items will be concurrently written to all healthy nodes. The write will not return until:
	* All nodes have been written to and responded, or timed out
* If any node responds with conditional write fail:
	* The call will return with conditional write fail
	* The value will be re-read from that node and unconditionally written to all healthy nodes
		* The call will not return until all nodes have responded or timed out on the second write

### Reading

* If no healthy nodes are available, the client will return an error.
* Two random healthy nodes are selected for reads.
* When both nodes return a cache miss, the response is a cache miss.
* If one node returns a cache miss and another node returns a hit, the value will be written to the missing node
* If both nodes timeout or error, they are marked unhealthy the read is restarted. 

### Health checks

* Health checks occur on all nodes periodically (default 10 seconds), and as part of any node operation
* A node health check will pass if:
	* The node responds to a GET for a random string with a cache miss within a timeout (100ms)
* A node health check will fail if:
	* The node fails to respond to any operation within a timeout (100ms)
	* The node responds with a Server Error
