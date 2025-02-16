#### &#x1F4DA; [Bookshelf](../)
#### &#x1F4DC; [Contents](./README.md#contents)
#### &#x1F448; [Prev](./Ch09_ZooKeeper_Internals.md)

## Chapter 10: Running ZooKeeper

- When an ensemble has enough ZooKeeper servers to start processing requests, we call the set of servers a quorum. Of course, we never want there to be two disjoint sets of servers that can process requests, or we would end up with split brain. We can avoid the split-brain problem by requiring that all quorums have at least a majority of servers. (Note: half of the servers do not constitute a majority; you must have greater than half the number of servers to have a majority.)

#### &#x1F4DA; [Bookshelf](../)
#### &#x1F4DC; [Contents](./README.md#contents)
#### &#x1F448; [Prev](./Ch09_ZooKeeper_Internals.md)