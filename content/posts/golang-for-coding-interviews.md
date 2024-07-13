+++
title = 'Golang for Coding Interviews'
date = 2024-07-12T21:31:35-07:00
draft = true
+++

Are you planning to use Go for solving coding problems in an upcoming interview? If so, you might have realized that the language feels a little "unergonomic" for quickly implementing algorithms under the time constraints imposed by an interview.

Unlike the real world, in a coding interview lack of conciseness and excess boilerplate gets in the way of finishing the solution in time and being able to communicate your ideas with the interviewer effectively.

In this article, I've gathered a few patterns I've found useful and effective when attempting coding interviews or Leetcode problems in Go.

## Who is this article really for?

This article needs to cover a lot of ground, so I will be restricting the scope significantly. 

This article IS NOT:
- A general introduction to Go (see [A tour of Go](https://go.dev/tour/welcome/1) instead)
- Example of good Go code (in fact a lot of it is rather inelegant and not ideal for production Go code)
- A general introduction to data structures and algorithms (see this excellent course by Robert Sedgewick & Kevin Wayne: [Part 1](https://www.coursera.org/learn/algorithms-part1), [Part 2](https://www.coursera.org/learn/algorithms-part2) instead)
- A guide to getting better at Leetcode (see [neetcode.io](https://neetcode.io) instead)

This article will be useful if:
- You already have a reasonable understanding of common algorithms and data structures
- You have been programming in Go for a while

## Why would someone attempt a coding interview in Go?

That's a fair question.

To be honest, it would only take a day or two to brush up on enough Python to be able to implement a vast majority of solutions you might come up with in an interview *(note: I am only talking about "implementing" which is the easy part unlike coming up with the solution in the first place)*. And in spite of my vested interest in having you read the rest of this article, if you have more than 3-4 weeks until your interview, I would urge you to reconsider and not use Go. It will save you a lot of time and headache.

I think you should only use Go for these interviews if:
1. The role specifically requires fluency in Go
2. You do not have the time or patience to brush up on Python/Java

## Challenges

As much as I love Go, here's what makes it annoying to use in interviews:

1. Verbosity (e.g. lack of Python-like list comprehension)
2. Boilerplate (e.g. try using the built-in Heap implementation)
3. Interviewer confusion (e.g. the concept of slices, appending to slices, reading from maps, short variable names, etc. are things that can appear really strange to interviewers unfamiliar with Go's syntax and conventions)

> [!TIP] Proactive clarification
> In my experience, it's better to **explicitly call out** when you're using Go syntax that is different from syntax of other languages. You could say something along the lines of *"This might look strange, but in Go, we do XYZ because ..."*. Most interviewers are reasonable and understand that each language has its own quirks and conventions, but if you don't call these out, they might walk away with an incorrect understanding of what your code does.

## Overview

There's roughly 3 parts to this:

1. Useful builtins and standard library function and 1D/2D arrays
2. Basic data structures like stacks, queues, graphs and related algorithms
3. Slightly advanced, but incredibly common algorithms and data structures like heaps, tries, topological sort, etc.

## Useful tips and patterns
### Math

The `math` library is pretty extensive, but for most interview questions, you only need to remember a small handful of functions.

```go
// min and max are builtins that take a variable number of args.
mn := min(a, b, c)
mx := max(a, b, c)

// Remember, these builtin functions do not work with slices.
// Also, min(someSlice...) doesn't work
// For slices, use the slices.Min / slices.Max functions which work with
// slices where the elements are of type cmp.Ordered
mn := slices.Min(someSlice)

// You often need square root of numbers
// Remember this takes float64 and returns float64
x := math.Sqrt(124.56)

// You might often need to round numbers
x := math.Ceil(y) // float64 -> float64
x := math.Floor(y) // float64 -> float64
```

Some other useful ones:

```go
func Abs(x float64) float64
func Log(x float64) float64
func Log10(x float64) float64
func Log2(x float64) float64
func Max(x, y float64) float64
func Min(x, y float64) float64
func Pow(x, y float64) float64
func Pow10(n int) float64
```

You'll rarely need the remaining functions in the `math` library, so I wouldn't bother remembering them. I can always implement it myself in a pinch.

Some of these methods (e.g.  `Abs`) might not actually be useful because you typically need `Abs` for `int` not `float64`. In such situations, simply implement a quick 3-line version for the `int` type.

### Strings

You only need to remember a handful of functions from the [`strings`](https://pkg.go.dev/strings) library:

```go
func Compare(a, b string) int
func Contains(s, substr string) bool
func HasPrefix(s, prefix string) bool
func HasSuffix(s, suffix string) bool
func Index(s, substr string) int
func Join(elems []string, sep string) string
func Replace(s, old, new string, n int) string
func ReplaceAll(s, old, new string) string
func Split(s, sep string) []string
func SplitN(s, sep string, n int) []string
func ToLower(s string) string
func ToUpper(s string) string
func Trim(s, cutset string) string
func TrimSpace(s string) string
```

You will rarely need anything else.

Sometimes you need to deal with runes (e.g. convert to equivalent ASCII value). Here are some useful conversions you will need:

```go
// Rune to ASCII
int(r)

// Scaled to 0-25 for lower case letters
// E.g. 'a' -> 0, 'b' -> 1, etc.
// This assumes input is in range a-z
int(int(r) - 'a')

// Convert 0-25 to rune
// 0 -> 'a', 1 -> 'b', etc.
rune('a' + index)

// Rune to string
string(r)
```

Convert between `int` and `string`:

```go
// Int to string
strconv.Itoa(i)

// String to int
i, ok := strconv.Atoi(s)

// base can be 0 (base is inferred from prefix)
// bitSize 0 implies int, 8 implies int8, and so on for 16, 32 and 64.
func ParseInt(s string, base int, bitSize int) (i int64, err error)
func ParseUint(s string, base int, bitSize int) (uint64, error)
```

Sometimes you need to build up strings incrementally piece by piece. Since strings are immutable, appending to a result string using the `+=` operator would not be optimal because it would create a lot of intermediate strings. Since this is the case is most languages (e.g. Java) most interviewers would be expecting you to use some construct that allows building up strings incrementally.

```go
var b strings.Builder

// strings.Builder implements io.Writer
for i := 3; i >= 1; i-- {
	fmt.Fprintf(&b, "%d...", i)
}

// It also has a method to directly write strings
b.WriteString("ignition")

// Construct the final string at the end
fmt.Println(b.String())
```

### "Arrays"

A vast majority of problems deal with 1D and 2D arrays. So you'll be defining them all the time.

(Well, as usual we are defining "slices" not "arrays". But, if your interviewer is not familiar with Go, this distinction would be difficult to explain quickly.)

```go
Rows := 10
Cols := 5

// 1D slice
arr := make([]int, Cols)

// 2D slice
matrix := make([][]int, Rows)
for r:=0; r<Rows; r++ {
	matrix[r] = make([]int, Cols)
}
```

> [!TIP] Proactive clarification
> Depending on the type of slice you create, you might be relying on the fact that the elements in the slice by default are the **zero** value of that particular type. Do let your interviewer know about this behavior. It can be somewhat surprising, especially when the type is a `struct` and the zero value is usually useable without having to construct each item.
> This is different compared to Java where `null` values are not useable at all.

### Binary search / Bisection

When you need to binary search a sorted array, the binary search functions from the `slices` package are incredibly useful:

```go
// returns the position where target is found, or the position where target would appear in the sort order; it also returns a bool saying whether the target is really found in the slice. 

func BinarySearch[S ~[]E, E cmp.Ordered](x S, target E) (int, bool)
func BinarySearchFunc[S ~[]E, E, T any](x S, target T, cmp func(E, T) int) (int, bool)
```

The second one is especially useful when you're dealing with a non-primitive data type:

```go
type Person struct {Id int, Name string}

persons := []Person{...}

// Notice how the target you are searching for does not need to be of the
// same type as the slice. This provides a lot of flexibility.
pIndex, ok := slices.BinarySearchFunc(persons, 5, func(p Person, id int) int {
	return cmp.Compare(p.Id, id)
})
```

When the element appears multiple times in the slice, the default implementation returns the leftmost index. But, what if you want it to return the index of the right most matching element? (In python, you have two separate functions `bisect_left` and `bisect_right` for this)

If you force the cmp function to return `-1` instead of `0` when the two elements are equal, it will end up returning the index of the item immediately to the right of the rightmost match.

```go
rb, _ := slices.BinarySearchFunc(nums, target, func(n1, n2 int) int {
	if n1 == n2 {
		return -1
	}
	return n1-n2
})

// IMPORTANT, since this will return the index to the right of the rightmost
// match.
rb = rb-1
```

### Bitwise operations

These are straight-forward, but some operations like **negation** use different symbols than Python.

| Operator | Meaning                                                                |
| -------- | ---------------------------------------------------------------------- |
| &        | And                                                                    |
| \|       | Or                                                                     |
| ^        | XOR (and also negation when used with a single operand, e.g. `y = ^x`) |
| <<       | Left shift                                                             |
| >>       | Right shift                                                            |
| &^       | And not (`x &^ y` means x AND (NOT y))                                 |
|          |                                                                        |

### Stack

This not being built-in is really a drag. But you can implement a barebones one using slices like this:

```go
type stack[T any] []T

func (s *stack[T]) push(x T) {
	*s = append(*s, x)
}

func (s *stack[T]) pop() (T, bool) {
	if len(*s) == 0 {
		var zero T
		return zero, false
	}
	top := (*s)[len(*s)-1]
	(*s) = (*s)[:len(*s)-1]
	return top, true
}

func (s stack[T]) len() int {
	return len(s)
}
```

This implementation uses generics. If you are forced to use a platform like coderpad that runs an old version of Go without generics, you can drop the type parameter and replace it with a concrete type.

Remember, since appending and removing from the slice returns a new slice value, we need to use a pointer to a slice to make this work correctly. This can be tricky to remember under pressure, so you're better off committing this (or a similar) implementation to memory.

### Queue

This is fairly similar to the stack implementation, but we chop off elements from the front.

```go
type queue[T any] []T

func (s *queue[T]) push(x T) {
	*s = append(*s, x)
}

func (s *queue[T]) pop() (T, bool) {
	if len(*s) == 0 {
		var zero T
		return zero, false
	}
	top := (*s)[0]
	(*s) = (*s)[1:]
	return top, true
}

func (s queue[T]) len() int {
	return len(s)
}
```

There are some memory implications of repeatedly using `(*s) = (*s)[1:]`. The underlying array won't get garbage collected until enough elements are appended to the slice to force a resize/doubling of the underlying array. Ideally, you could use a circular buffer to implement the queue and avoid this problem. But realistically, this trivial implementation should suffice for a coding interview. If the interviewer is familiar with Go, you might want to clarify that you can use a circular buffer in a real implementation.

It is also trivial to implement a queue using channels. But I would avoid using Go specific features like channels in an interview unless the question is specifically related to implementing a multi-threading/concurrency.

### Trees

A tree node needs the value and left/right pointers to other nodes:

```go
type Node struct {
	Val   int
	Left  *Node
	Right *Node
}
```

To represent and pass around a tree you only need to pass the pointer to its root.

### Graphs

Graphs can be represented as adjacency lists (ideal for sparse graphs) and adjacency matrix (ideal for dense graphs). Typically in most interview problems, nodes on the graph are identified by some integer ID.

Adjacency lists can be represented as a map:

```go
graph := map[int][]int {
	0: []int{2, 3},
	1: []int{4},
	2: []int{0},
	3: []int{0, 4},
	4: []int{1, 3},
}
```

Adjacency matrixes can be represented as 2D slices:

```go
// This assumes edges are unweighted.
// For weighted edges, it can be a slice of int/float.
graph := [][]bool{
	{false, false, true, true, false},
	{false, false, false, false, true},
	{true, false, false, false, false},
	{true, false, false, false, true},
	{false, true, false, true, false},
}
```

### DFS

With adjacency list:
```go
// Example of using DFS to find all reachable nodes.
func reachable(graph map[int][]int, start int) []int {
	visited := make(map[int]bool)
	results := []int{}

	dfs(graph, start, visited, &results)
	return results
}

func dfs(graph map[int][]int, start int, visited map[int]bool, results *[]int) {
	if visited[start] {
		return
	}

	visited[start] = true
	(*results) = append((*results), start)

	for _, neighbor := range graph[start] {
		dfs(graph, neighbor, visited, results)
	}
}
```

With adjacency matrix:
```go
type coord struct{ x, y int }

// Example of using DFS to find all reachable cells from (i,j).
func reachable(grid [][]int, i, j int) []coord {
	visited := make(map[coord]bool)
	results := []coord{}

	dfs(grid, coord{i, j}, coord{len(grid), len(grid[0])}, visited, &results)
	return results
}

func dfs(grid [][]int, start, limits coord, visited map[coord]bool, results *[]coord) {
	if visited[start] {
		return
	}

	visited[start] = true
	(*results) = append((*results), start)

	for _, neighbor := range neighbors(start, limits) {
	    // IMPORTANT: Check whatever is the definition of "reachable"/
	    // connectectivity between two cells depending on the problem.
		if grid[neighbor.x][neighbor.y] == 1 {
			dfs(grid, neighbor, limits, visited, results)
		}
	}
}

func neighbors(point, limits coord) []coord {
	ret := []coord{}
	for _, offset := range []coord{
		{-1, 0},
		{1, 0},
		{0, -1},
		{0, 1},
	} {
		neighbor := coord{point.x + offset.x, point.y + offset.y}
		if neighbor.x < 0 || neighbor.x >= limits.x {
			continue
		}
		if neighbor.y < 0 || neighbor.y >= limits.y {
			continue
		}
		ret = append(ret, neighbor)
	}

	return ret
}
```

### BFS

With adjacency list:
```go
// Example of using BFS to find all reachable nodes.
func reachable(graph map[int][]int, start int) []int {
	visited := make(map[int]bool)
	results := []int{}

	// This is the queue implementation from earlier in this article
	q := &queue[int]{}
	q.push(start)

	for q.len() > 0 {
		node, _ := q.pop()
		if visited[node] {
			continue
		}

		results = append(results, node)
		visited[node] = true

		for _, neighbor := range graph[node] {
			if visited[neighbor] {
				continue
			}
			q.push(neighbor)
		}
	}
	
	return results
}
```

With adjacency matrix:
```go
type coord struct{ x, y int }

// Example of using BFS to find all reachable cells from (i,j).
func reachable(grid [][]int, i, j int) []coord {
	// MxN the dimentions of the grid.
	limits := coord{len(grid), len(grid[0])}

	visited := make(map[coord]bool)
	results := []coord{}

	// This is the queue implementation from earlier in this article
	q := &queue[coord]{}
	q.push(coord{i, j})

	for q.len() > 0 {
		cell, _ := q.pop()
		if visited[cell] {
			continue
		}

		visited[cell] = true
		results = append(results, cell)

		for _, neighbor := range neighbors(cell, limits) {
			if visited[neighbor] {
				continue
			}

			// IMPORTANT: Check whatever is the definition of "reachable"/
		    // connectectivity between two cells depending on the problem.
			if grid[neighbor.x][neighbor.y] == 1 {
				q.push(neighbor)
			}
		}
	}

	return results
}

func neighbors(point, limits coord) []coord {
	ret := []coord{}
	for _, offset := range []coord{
		{-1, 0},
		{1, 0},
		{0, -1},
		{0, 1},
	} {
		neighbor := coord{point.x + offset.x, point.y + offset.y}
		if neighbor.x < 0 || neighbor.x >= limits.x {
			continue
		}
		if neighbor.y < 0 || neighbor.y >= limits.y {
			continue
		}
		ret = append(ret, neighbor)
	}

	return ret
}
```

### Heap

Heaps show up in a vast majority of problems (e.g. job scheduling, greedy exploration of paths, etc.). It's also a core component of many other algorithms (e.g. Dijkstra's algorithm).

The problem with heaps in Go, is that it requires a non-trivial amount of boilerplate. You need to define a custom type and implement 5 different methods to make it implement `heap.Interface`. This is in sharp contrast to Python where you can use a regular list as a heap and throw tuples of `(priority, object)` into it using the helper methods in the `heap` library.

```go
import "container/heap"

// You need to define a custom type like this.
type IntHeap []int

// And implement these. 5 methods.
func (h IntHeap) Len() int           { return len(h) }
func (h IntHeap) Less(i, j int) bool { return h[i] < h[j] }
func (h IntHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }

func (h *IntHeap) Push(x any) {
	// Push and Pop use pointer receivers because they modify the slice's length,
	// not just its contents.
	*h = append(*h, x.(int))
}

func (h *IntHeap) Pop() any {
	old := *h
	n := len(old)
	x := old[n-1]
	*h = old[0 : n-1]
	return x
}

// This example inserts several ints into an IntHeap, checks the minimum,
// and removes them in order of priority.
func main() {
	h := &IntHeap{2, 1, 5}
	heap.Init(h)
	heap.Push(h, 3)
	fmt.Printf("minimum: %d\n", (*h)[0])
	for h.Len() > 0 {
		fmt.Printf("%d ", heap.Pop(h))
	}
}
```

One nice feature of the Go `heap` library is that it has a `Fix` method that allows you to re-adjust the heap after you modify the priority of an item on the heap. A really convenient way to implement this is by storing the index of the element within the element and updating it in the `Swap` method (this example comes directly from the examples in the `heap` library). You need this index so that you can call `Fix` after updating the item.

```go
type Item struct {
	value    string
	priority int
	
	index int // IMPORTANT: The index of the item in the heap.
}

// The type we will use as the Heap
type PriorityQueue []*Item

// Omitting the other 4 methods, except Swap.

// Use Swap for maintaining the index value in the item.
func (pq PriorityQueue) Swap(i, j int) {
	pq[i], pq[j] = pq[j], pq[i]
	pq[i].index = i
	pq[j].index = j
}

// Update modifies the priority and value of an Item in the queue.
func (pq *PriorityQueue) update(item *Item, value string, priority int) {
	item.value = value
	item.priority = priority
	heap.Fix(pq, item.index) // This is why we need to track the index.
}
```

### Tries

Tries are useful for a variety of problems that involve searching for words based on prefixes or patterns.

```go
type TrieNode struct {
    isWord bool
    children map[byte]*TrieNode
}

func newNode() *TrieNode {
    return &TrieNode{
        children: make(map[byte]*TrieNode),
    }
}

func buildTrie(root *TrieNode, word string) {
    if word == "" {
        root.isWord = true
        return
    }

    first := word[0]
    remaining := word[1:]

    child, ok := root.children[first]
    if !ok {
        child = newNode()
        root.children[first] = child
    }

    buildTrie(child, remaining)
}
```

### (Sidenote) Passing mutable slices to functions

Sometimes it is convenient to be able to pass a slice to a function (maybe recursively) for the function to append results into it (e.g. recursive DFS based topological sort implementation that inserts nodes into a `result` slice).

In most languages lists are passed by reference and appends to the list are trivial from inside the function. This is not the case for Go because of how slices work in Go.

```go
func someFunction(results []int) {
	...
	...
	// Here `results` is actually just the slice header passed by value.
	// When we append to the slice, the underlying array might be replaced
	// with a new one depending on whether there is capacity remaining.
	// So we usually capture the return value from `append`
	results = append(results, 10)
	// We are not actually returning back the return value from append.
	// So the caller is still pointing to the old slice header.
	...
}
```

A lot of algorithms are simpler to implement recursively when we can mutate slices within functions.

One way to make this work is to replace the slice with a **pointer to a slice** (note: different from a slice of pointers).

```go
func someFunction(results *[]int) {
	...
	...
	// This works
	(*results) = append((*results), 10)
	...
}
```

### Topological sort

Here's a simple topological sort implementation that also returns an error when there is a cycle in the graph.

```go
func topoSort(graph map[int][]int, goal int) ([]int, error) {
	visited := make(map[int]bool)
	currentPath := make(map[int]bool)
	results := []int{}

	if err := dfs(graph, goal, visited, currentPath, &results); err != nil {
		return nil, err
	}
	return results, nil
}

func dfs(graph map[int][]int, start int, visited, currentPath map[int]bool, results *[]int) error {
	if currentPath[start] {
		return errors.New("cycle detected")
	}

	if visited[start] {
		return nil
	}

	visited[start] = true
	currentPath[start] = true
	defer func() {
		currentPath[start] = false
	}()

	for _, neighbor := range graph[start] {
	    // IMPORTANT TO CATCH AND RETURN IF ERR HERE
		if err := dfs(graph, neighbor, visited, currentPath, results); err != nil {
			return err
		}
	}

	(*results) = append((*results), start)
	return nil
}
```

### Union-Find

Union-find is a really useful data structure that allows "connecting" two nodes and then quickly  determining if any two nodes are connected directly or indirectly. It is used internally in the implementation of many different algorithms.

This is a really interesting data structure, but explaining how it works is outside the scope of this article. I would strongly recommend watching Robert Sedgewick's video on the topic. See [this](https://www.coursera.org/learn/algorithms-part1) course and these [slides](https://algs4.cs.princeton.edu/lectures/keynote/15UnionFind-2x2.pdf).

```go
type UF struct {
	roots []int
}

func Init(N int) *UF {
	roots := make([]int, N)
	for i := range N {
		roots[i] = i
	}
	return &UF{
		roots: roots,
	}
}

func (u *UF) root(i int) int {
	for i != u.roots[i] {
		u.roots[i] = u.roots[u.roots[i]] // Path compression
		i = u.roots[i]
	}
	return i
}

func (u *UF) Union(i, j int) {
	r1 := u.root(i)
	r2 := u.root(j)
	u.roots[r2] = r1
}

func (u *UF) IsConnected(i, j int) bool {
	return u.root(i) == u.root(j)
}
```

### Randomness

Use `math/rand` package. In most situations you'll only need these functions:

```go
func Float32() float32  // In interval [0.0, 1.0)
func Float64() float64  // In interval [0.0, 1.0)
func Int() int  // Random int
func Intn(n int) int  // Random int in interval [0,n)
```

### Concurrency

Unless you are interviewing for a position that requires specific knowledge of Go, it is unlikely anyone would ask you a question about Go's concurrency primitives.

Rather, the question would be something more generic (e.g. write a multi-threaded web crawler).

For these questions, it is useful to get familiar with advanced concurrency patterns in your language of choice. It is pointless to cover this here since it is already explained really well in the sources below:

1. [Go concurrency patterns](https://go.dev/talks/2012/concurrency.slide#1)
2. [Go advanced concurrency patterns](https://go.dev/talks/2013/advconc.slide#1)
3. [Pipelines and cancellation](https://go.dev/blog/pipelines)

And for a more concrete example see: https://medium.com/@mikatal/concurrent-web-crawler-using-go-6ce9e4745527.

## Next steps

I know we all hate Leetcode style programming interviews. But there's really no escaping it at the moment.

I would **strongly** recommend going over to [Leetcode](https://leetcode.com/) and practicing the most popular problems in Go. You can also use [Neetcode's roadmap](https://neetcode.io/roadmap) as a good starting point.