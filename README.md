# Original BFS Implementation

This branch contains the original Breadth-First Search pathfinding implementation based on concepts from *Beginning Game AI with Unity* by Sebastiano M. Cossu.

The algorithm uses:

- A `Queue<Node>` to explore the grid.
- A `List<Node>` to track visited nodes.
- The `Depth` property to reconstruct the path from the goal back to the start.

This version is kept as the starting point for comparing later BFS improvements.
