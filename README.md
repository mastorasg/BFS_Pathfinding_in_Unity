# Unity BFS Grid Pathfinding

A Unity project that demonstrates grid-based pathfinding with the Breadth-First Search (BFS) algorithm.

The project creates a grid of walkable nodes and walls, finds the shortest available path between a start node and a goal node, and moves an agent through the calculated route. It is based on concepts from *Beginning Game AI with Unity: Programming Artificial Intelligence with C#* by Sebastiano M. Cossu, with additional BFS improvements implemented during the project. [Source book listing](https://www.academia.edu/79828958/Beginning_Game_AI_with_Unity_Programming_Artificial_Intelligence_with_C_Sebastiano_M_Cossu_)

## Features

- Generates a predefined grid of navigation nodes.
- Creates visual waypoints using a Unity prefab.
- Supports walkable nodes and blocked wall nodes.
- Finds node neighbours in four directions:
  - Up
  - Down
  - Left
  - Right
- Calculates the shortest path using Breadth-First Search.
- Moves and rotates an agent toward each waypoint in the generated path.
- Uses Unity materials to visually distinguish walls and the goal node.
- Includes three BFS implementations:
  - Original depth-based BFS.
  - Improved depth-based BFS using `HashSet<Node>`.
  - Parent-map BFS using `Dictionary<Node, Node>`.

## Controls

| Key | Action |
|---|---|
| `Return` / `Enter` | Reset the agent to the start node, calculate the shortest path, and begin movement toward the goal |

## Project Setup

1. Create a new Unity scene.
2. Create an empty GameObject for the moving agent.
3. Attach the `GridWP` script to the agent.
4. Create a waypoint prefab, such as a Cube or Sphere.
5. Assign the prefab to `Prefab Waypoint` in the Inspector.
6. Create two materials:
   - A wall material, assigned to `Wall Mat`.
   - A goal material, assigned to `Goal Mat`.
7. Enter Play mode.
8. Press `Return` to calculate and follow the path.

> The waypoint prefab must contain a `Renderer` component because the script changes its material.

## How It Works

The grid is represented by a two-dimensional `Node[,]` array. Each node contains:

- Whether the node is walkable.
- Its waypoint GameObject.
- A list of adjacent walkable nodes.
- A depth value for the depth-based BFS implementations.

```csharp
public class Node
{
    private int depth;
    private bool walkable;
    private GameObject waypoint = new GameObject();
    private List<Node> neighbors = new List<Node>();

    public int Depth { get => depth; set => depth = value; }
    public bool Walkable { get => walkable; set => walkable = value; }
    public GameObject Waypoint { get => waypoint; set => waypoint = value; }
    public List<Node> Neighbors { get => neighbors; set => neighbors = value; }

    public Node()
    {
        depth = -1;
        walkable = true;
    }

    public Node(bool walkable)
    {
        depth = -1;
        this.walkable = walkable;
    }
}
```

A node created with `new Node()` is walkable. A node created with `new Node(false)` represents a wall.

The script checks every node's valid neighbours while staying inside the bounds of the grid. Diagonal movement is not supported, so an agent can only travel vertically or horizontally.

## BFS Implementations

### 1. Original depth-based BFS

The original version uses:

- `Queue<Node>` to process nodes in BFS order.
- `List<Node>` to track visited nodes.
- The `Depth` property to identify how far each node is from the start.
- Depth comparison to reconstruct the final path.

```csharp
List<Node> BFS(Node start, Node end)
{
    Queue<Node> toVisit = new Queue<Node>();
    List<Node> visited = new List<Node>();

    Node currentNode = start;
    currentNode.Depth = 0;
    toVisit.Enqueue(currentNode);

    List<Node> finalPath = new List<Node>();

    while (toVisit.Count > 0)
    {
        currentNode = toVisit.Dequeue();

        if (visited.Contains(currentNode))
            continue;

        visited.Add(currentNode);

        if (currentNode.Equals(end))
        {
            while (currentNode.Depth != 0)
            {
                foreach (Node n in currentNode.Neighbors)
                {
                    if (n.Depth == currentNode.Depth - 1)
                    {
                        finalPath.Add(currentNode);
                        currentNode = n;
                        break;
                    }
                }
            }

            finalPath.Reverse();
            break;
        }

        foreach (Node n in currentNode.Neighbors)
        {
            if (!visited.Contains(n) && n.Walkable)
            {
                n.Depth = currentNode.Depth + 1;
                toVisit.Enqueue(n);
            }
        }
    }

    return finalPath;
}
```

This version is useful for understanding how BFS explores a graph in layers. The start node has a depth of `0`, its neighbours have a depth of `1`, their neighbours have a depth of `2`, and so on.

When BFS reaches the goal, the algorithm reconstructs the path by repeatedly finding a neighbour with a depth one lower than the current node.

### 2. Improved BFS with HashSet and depth

The first improved version keeps depth-based path reconstruction but replaces the visited list with `HashSet<Node>`.

```csharp
List<Node> BFS(Node start, Node end)
{
    Queue<Node> toVisit = new Queue<Node>();
    HashSet<Node> visited = new HashSet<Node>();
    List<Node> finalPath = new List<Node>();

    start.Depth = 0;
    toVisit.Enqueue(start);
    visited.Add(start);

    while (toVisit.Count > 0)
    {
        Node currentNode = toVisit.Dequeue();

        if (currentNode.Equals(end))
        {
            Node curr = currentNode;

            while (curr != null)
            {
                finalPath.Add(curr);

                if (curr.Depth == 0)
                    break;

                Node previous = null;

                foreach (Node n in curr.Neighbors)
                {
                    if (n.Depth == curr.Depth - 1)
                    {
                        previous = n;
                        break;
                    }
                }

                curr = previous;
            }

            finalPath.Reverse();
            return finalPath;
        }

        foreach (Node n in currentNode.Neighbors)
        {
            if (!visited.Contains(n) && n.Walkable)
            {
                n.Depth = currentNode.Depth + 1;
                visited.Add(n);
                toVisit.Enqueue(n);
            }
        }
    }

    return finalPath;
}
```

#### Improvements in this version

- `HashSet<Node>` provides faster average lookup than `List<Node>`.
- Nodes are marked as visited when they are added to the queue.
- Marking nodes during enqueueing prevents duplicate queue entries.
- The path is returned immediately after the destination is found.
- The condition `curr.Depth == 0` clearly identifies when reconstruction has reached the start node.

This version still relies on node depth to find the previous node during path reconstruction.

### 3. BFS with a parent dictionary

The final version uses a dictionary to store the parent of every discovered node.

```csharp
List<Node> BFS(Node start, Node end)
{
    Queue<Node> toVisit = new Queue<Node>();
    HashSet<Node> visited = new HashSet<Node>();
    Dictionary<Node, Node> parentMap = new Dictionary<Node, Node>();
    List<Node> finalPath = new List<Node>();

    toVisit.Enqueue(start);
    visited.Add(start);

    while (toVisit.Count > 0)
    {
        Node currentNode = toVisit.Dequeue();

        if (currentNode.Equals(end))
        {
            Node curr = end;

            while (curr != null)
            {
                finalPath.Add(curr);
                parentMap.TryGetValue(curr, out curr);
            }

            finalPath.Reverse();
            return finalPath;
        }

        foreach (Node n in currentNode.Neighbors)
        {
            if (n.Walkable && !visited.Contains(n))
            {
                visited.Add(n);
                parentMap[n] = currentNode;
                toVisit.Enqueue(n);
            }
        }
    }

    return finalPath;
}
```

#### Why use a parent dictionary?

When BFS discovers a new node, it saves the node from which it was reached:

```csharp
parentMap[n] = currentNode;
```

For example, if BFS reaches node `B` from node `A`, the dictionary stores:

```text
parentMap[B] = A
```

When the destination is found, the algorithm follows those parent references:

```text
Goal -> Previous node -> Previous node -> Start
```

The list is then reversed so the agent can move from the start node to the goal node.

This is the recommended implementation because it explicitly stores the exact route discovered by BFS. It also removes the need to use or reset the `Depth` property.

## Comparison of BFS Versions

| Version | Visited collection | Path reconstruction | Uses depth | Main purpose |
|---|---|---|---|---|
| Original BFS | `List<Node>` | Finds a neighbouring node with lower depth | Yes | Learning the basic BFS process |
| Improved BFS | `HashSet<Node>` | Finds a neighbouring node with lower depth | Yes | Avoiding duplicate queue entries and improving lookups |
| Parent-map BFS | `HashSet<Node>` | Follows saved parent references | No | Clear and robust path reconstruction |

## Agent Movement

After BFS returns a list of nodes, the agent moves toward each node's waypoint.

```csharp
Vector3 direction = goal - transform.position;

transform.rotation = Quaternion.Slerp(
    transform.rotation,
    Quaternion.LookRotation(direction),
    Time.deltaTime * rotSpeed
);

transform.Translate(0, 0, speed * Time.deltaTime);
```

The agent moves to the next waypoint after it reaches the current one within the configured accuracy distance.

```csharp
if (curNode < path.Count - 1)
{
    curNode++;
}
```

## Concepts Practiced

- Unity GameObjects, prefabs, materials, and transforms.
- C# classes, constructors, properties, and method overriding.
- Two-dimensional arrays with `Node[,]`.
- Generic collections:
  - `List<T>`
  - `Queue<T>`
  - `HashSet<T>`
  - `Dictionary<TKey, TValue>`
- Graph representation using neighbours.
- Breadth-First Search.
- Shortest-path search in an unweighted graph.
- Parent-based path reconstruction.
- Agent movement and rotation in Unity.

## Possible Improvements

- Allow the user to click to select start and goal nodes.
- Display nodes as they are visited by BFS.
- Colour the final calculated path.
- Add diagonal movement.
- Add weighted terrain and implement Dijkstra's algorithm.
- Implement A* pathfinding with a heuristic.
- Use a Unity Tilemap instead of a manually defined grid.
- Generate random walls or mazes.
- Animate the search using a coroutine.
- Add multiple agents using the same navigation grid.

## Learning Source

This project was created as a learning exercise based on ideas from:

> Sebastiano M. Cossu, *Beginning Game AI with Unity: Programming Artificial Intelligence with C#*.

The project expands on the original depth-based BFS approach with two alternative implementations: an improved `HashSet<Node>` version and a `Dictionary<Node, Node>` parent-map version.
