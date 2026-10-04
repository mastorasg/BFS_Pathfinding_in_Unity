# BFS with HashSet and Depth

This branch contains a small improvement to the original Breadth-First Search implementation.

Changes from the original version:

- Replaces `List<Node>` with `HashSet<Node>` for visited nodes.
- Marks nodes as visited when they are added to the queue.
- Prevents duplicate nodes from being added to the BFS queue.
- Keeps the original `Depth`-based path reconstruction.

This version improves the BFS visitation logic while preserving the original algorithm structure.
