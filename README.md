Maximum Matching Program
========================

A **graph** consists of vertices connected by edges. For example, the following image contains a graph with ten vertices, labelled from *a* to *j*:

![Graph](https://github.com/benstamour/maximum-matching/blob/main/pictures/graph1.PNG?raw=true "Graph")

If two vertices are connected by an edge, they are **adjacent** to each other. For example, in the above graph, vertex *c* is adjacent to vertices *f* and *j*. Similarly, if two edges share a common vertex, the edges are also adjacent to each other. The edges connecting vertex *c* to vertices *f* and *j* are adjacent to each other.

A **matching** is a set of pairwise non-adjacent edges; for example, in the above graph, a matching could include the edge connecting vertices *b* and *g* and the edge connecting *c* and *j*, since each vertex is connected to at most one of the edges in the matching.

However, if we add the edge connecting *c* and *f*, then we no longer have a matching, because vertex *c* will be connected to two edges in the matching, and we would therefore have two adjacent edges.

A **maximal matching** is a matching where no more edges can be added to the matching without violating the rule that each vertex be connected to at most one edge. In the following image, the purple edges create a maximal matching.

![Graph with Matching](https://github.com/benstamour/maximum-matching/blob/main/pictures/graph2.PNG?raw=true "Graph with Matching")

This is a maximal matching because we cannot add another edge to the matching without having at least one vertex connected to more than one edge in the matching (the purple edges). Note that it is not necessary for all vertices to be connected to an edge in the matching in order for it to be maximal - in the above image, vertices *a* and *f* are not connected to any edge in the maximal matching.

A **maximum matching** is a matching that contains the maximum number of edges possible. A maximum matching must be maximal, but a maximal matching may not be a maximum matching.

You may also notice that out of vertices *a* through *e*, none of them are connected to each other. Likewise, no vertex from *f* through *j* is connected to another vertex in that range. When this happens, the graph is called **bipartite**. A bipartite graph contains two disjoint sets of vertices *X* and *Y* where each edge in the graph connects a vertex in *X* with a vertex in *Y*. We can divide the above graph into two sets below, which shows it to be bipartite:

![Graph Shown as Bipartite](https://github.com/benstamour/maximum-matching/blob/main/pictures/graph3.PNG?raw=true "Graph Shown as Bipartite")

Here, we have also renamed vertices *a* through *e* to be *x*1 through *x*5, while vertices *f* through *j* have been renamed to *y*1 through *y*5. It can be seen that each edge in the graph connects a vertex in *X* with a vertex in *Y*.

Now, using the code below, we can find a maximum matching for any bipartite graph. We first define classes for matchings and vertices, along with the `Alg` class, which contains subsets of *X* and *Y* and where we will be using our augmenting path algorithm `augpath()` to determine our maximum matching.

```
# Imports
from array import *
import copy

# Class for matchings
class Matching(object):                 # each Matching object contains a set of edges and the number of edges in the matching
    def __init__(self, edgeSet, size):  # sets all attributes of newly created Matching object
        self.edgeSet = edgeSet
        self.size = size

# Class for vertices
class Vertex(object): # each Vertex object contains a number corresponding to its place in the graph and adjacency matrix, an indicator of whether it has been marked,
                      # and the vertex it was reached from (if applicable) when using augmenting path algorithm
    def __init__(self, num, marked, reachedFrom):   # sets all attributes of newly created Vertex object
        self.num = num
        self.marked = marked
        self.reachedFrom = reachedFrom

# Class to be used with the augmenting path algorithm
class Alg(object):                  # each Alg object contains sets S (subset of X) and T (subset of Y) which are the sets of vertices reached
    def __init__(self, S, T):       # sets all attributes of newly created Alg object
        self.S = S
        self.T = T

    def augpath(self, G, M, U):     # implements augmenting path algorithm, where G is an array representing the graph, M is a matching, and U is the set of vertices in the graph.
        if M.size == 1:             # if the matching consists of just one edge, initialize S and T
            self.S = U
            self.T = []
            
        noUnmarkedVertex = True             # Boolean variable indicating whether S has an unmarked vertex
        for v in self.S:                    # checks whether S has an unmarked vertex
            if v.marked == False:
                noUnmarkedVertex = False
                break
        if noUnmarkedVertex == True:        # if S has no unmarked vertex, we have explored every vertex in S so M is a maximal matching and we return M
            return M
        else:
            reset = []                      # if we restart the algorithm again with j=0 (retaining certain edges), this will include the vertices to skip over
            for j in range(0, len(self.S)): # for each vertex in S
                x = self.S[j]               # set the Vertex object x be element j in S
                if x in reset:              # if x is in reset, it is already an endpoint of an edge in M that we do not want to change
                    continue
                if x.marked == False:       # checks if x is an unmarked vertex
                    c = False               # Boolean variable indicating whether x is an endpoint of an edge in M
                    for e in M.edgeSet:     # checks if x is already an endpoint of an edge in M
                        if e[0] == x.num:
                            c = True
                            break
                    if c:                   # if x is an endpoint of an edge in M, we end this iteration of the loop
                        continue
                    
                    Nx = []                     # to store the neighbours of x
                    for i in range(0, numY):    # for each vertex in the set Y
                        if G[x.num][i] > 0:     # if the vertex is adjacent to x, add it to Nx
                            Nx.append(i)
                            
                    for y in Nx:                # consider each y in Nx (note that the vertex y is just an integer variable here, not a Vertex object like x is)
                        saturated = False       # Boolean variable indicating whether y is saturated by M
                        for e in M.edgeSet:
                            if e[1] == y:       # if y is the endpoint of an edge already in M
                                s = True        # Boolean variables indicating whether y and the other endpoint of this edge are already in T and S respectively
                                t = True
                                for p in self.S:
                                    if p.num == e[0]:           # if the endpoint in X is already in S
                                        s = False
                                        p.reachedFrom = y       # records number of vertex from which p was reached
                                        break
                                for q in self.T:
                                    if q.num == e[1]:           # if a Vertex object with num = y is already in T
                                        t = False
                                        q.reachedFrom = x.num   # records number of vertex from which q was reached
                                        break
                                if s:                           # if the endpoint in X is not already in S, add this endpoint to S as a Vertex object
                                    self.S.append(Vertex(e[0], False, y))
                                if t:                           # if a Vertex object with num = y is not in T, create a Vertex object in T with num = y
                                    self.T.append(Vertex(e[1], False, x.num))
                                saturated = True
                                break
                        if saturated == True:                   # if y is saturated by M, end this iteration of the loop
                            continue
                        else:
                            b = False                           # Boolean variable indicating whether x is already saturated by M
                            for e in M.edgeSet:                 # for every edge in M, if x is saturated by M, we end this iteration of the loop
                                if e[0] == x.num:               # this is the second time we test for this because we may have just added an edge into M which saturates
                                                                # x and one of its neighbours in a previous iteration of the loop
                                    b = True
                                    M.edgeSet.remove(e)         # if x is the endpoint of an edge in M, replace this edge in M with a new edge connecting x and y
                                                                # this allows us to consider each y in Nx
                                    M.edgeSet.append([x.num, y])
                                    reset.append(x)             # we will go through every vertex with the algorithm again but we will skip over the vertex x because
                                                                # we want to keep this edge in M
                                    j = 0                       # resetting the value of j; we want to consider the previous x's again because one of those vertices may
                                                                # be adjacent to the other endpoint of e which is in Y
                                    for v in self.S:            # change every marked vertex in S that isn't in the reset list to unmarked
                                        if v not in reset:
                                            v.marked = False
                                    self.augpath(G, M, U)       # recursively call the algorithm again using this new matching
                                    break
                            if b == True:                       # if x is saturated by M, move on to the next iteration of the loop
                                continue
                            M.edgeSet.append([x.num, y])        # otherwise, add an edge to M containing the num value of the Vertex object x along with the value of y
                            M.size += 1                         # increase the size of the matching M by 1
                            self.augpath(G, M, U)               # recursively call the algorithm again to try to get a new edge to add to M
                x.marked = True                                 # after we have checked all neighbours of x, we mark x and move on to the next iteration of the loop
        return M                                                # if we exit the loop, return the matching M
```
With our classes defined, we now take input from the user to construct our bipartite graph using the following code:
```
# gets number of vertices in the sets X and Y from the user
numX = int(input("Enter the number of vertices in X: "))
numY = int(input("Enter the number of vertices in Y: "))
array = [[0 for j in range(0,numY)] for i in range(0, numX)]    # adjacency matrix of the graph

print("Order the vertices in X as x1 to x" + str(numX) + " and the vertices in Y from y1 to y" + str(numY) + ".\n")
print("For each vertex in X, list the vertices in Y that are adjacent to it, separated by spaces; for example, if y1 and y3 are adjacent to the given vertex, then you would enter '1 3'.")

# gets each value in the adjacency matrix from the user
for i in range(0,numX):
    yvals = input("Enter the values of each vertex in Y that is adjacent to x" + str(i+1) + ": ").split()
    for j in yvals:
        array[i][int(j)-1] = 1
print("\n********************\n")
```
We can run the above code and enter our bipartite graph from above as a sample input as follows:
```
Enter the number of vertices in X: 5
Enter the number of vertices in Y: 5

Now we will fill in the adjacency matrix.
Order the vertices in X as x1 to x5 and the vertices in Y from y1 to y5.

For each vertex in X, list the vertices in Y that are adjacent to it, separated by spaces; for example, if y1 and y3 are adjacent to the given vertex, then you would enter '1 3'.
Enter the values of each vertex in Y that is adjacent to x1: 4
Enter the values of each vertex in Y that is adjacent to x2: 2 5
Enter the values of each vertex in Y that is adjacent to x3: 1 5
Enter the values of each vertex in Y that is adjacent to x4: 2 4
Enter the values of each vertex in Y that is adjacent to x5: 1 3 4

********************
```
With our bipartite graph created, we now use our algorithm to determine a set of edges that form a maximum matching. Note that a graph may have multiple maximum matchings; this program will just find one of these matchings.
```
maxSize = 0                                 # size of the largest matching encountered so far will be stored here
maxMatching = None                          # the largest matching encountered so far will be stored here
U = []
for i in range(0, numX):
    U.append(Vertex(i, False, None))        # adds every vertex in X into U as a Vertex object
origU = copy.deepcopy(U)                    # creates copy of U to save this original version for later

for i in range(0, numX):                    # run algorithm for each edge in the graph
    for j in range(0, numY):
        if array[i][j] > 0:                 # tests if an edge connecting the corresponding vertices in X and Y exists
            edge = Matching([[i,j]], 1)     # creates a new instance of Matching containing one edge in its edgeSet
            if maxSize == 0:                # if this is the first edge encountered by the algorithm, this edge is the largest matching encountered so far
                maxSize = 1
                maxMatching = edge
            for v in U:                     # removes the vertex in U that is saturated by edge
                if v.num == i:
                    U.remove(v)
                    break
            A = Alg(U, [])                  # creates instance of Alg
            M = A.augpath(array, edge, U)   # runs augmenting path algorithm
            if M.size > maxSize:            # if the returned matching is larger than the current largest matching, the returned matching is the new largest matching
                maxSize = M.size
                maxMatching = M
            U = copy.deepcopy(origU)        # restores U to its original state by replacing it with the copy in preparation for next iteration of the loop           

print("A maximum matching of the graph contains the following " + str(maxSize) + " edges in the form [xa, yb],\n" +
      "where xa is vertex a in the set X and yb is vertex b in the set Y:\n")
for e in maxMatching.edgeSet:               # prints all edges in the maximum matching of the graph
    print("[x" + str(e[0] + 1) + ", y" + str(e[1] + 1) + "]")
```
The above code yields the following output:
```
A maximum matching of the graph contains the following 5 edges in the form [xa, yb],
where xa is vertex a in the set X and yb is vertex b in the set Y:

[x2, y5]
[x1, y4]
[x3, y1]
[x4, y2]
[x5, y3]
```
This gives us our result; if we colour the corresponding edges in our graph, we can see that each vertex is connected to at most one purple edge in our matching. In fact, each vertex is connected to exactly one purple edge, making the matching perfect. A perfect matching is guaranteed to be maximum, although a maximum matching may not necessarily be perfect.

![Graph with Maximum Matching](https://github.com/benstamour/maximum-matching/blob/main/pictures/graph4.PNG?raw=true "Graph with Maximum Matching")

To use this program with your own graph, you can download and run the `maxmatching.py` file with the command `python maxmatching.py`.
