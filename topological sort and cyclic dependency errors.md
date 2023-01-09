There is a good chance you've came across cyclic dependency errors while building applications in React, React Native, Angular, Vue, Go, NodeJS etc.
In some programming languages like Go, cyclic dependency is a compile time error that must be fixed before the program works but in React it's just a warning, which can cause unexpected behaviors in your application.

**In this article we'll be discussing cyclic dependency errors, what they are, how to avoid them and how the underlying algorithm (topological sort) works.**

##Understanding Cyclic Dependency Errors
Cyclic dependency errors occur when two or more modules, classes, or objects depend on each other in a circular manner. This creates a situation in which the modules, classes, or objects cannot be used or instantiated because their dependencies are not satisfied.

What does **depend on each other in a circular manner** mean?
To under this let's first talk about how files are built (declaring & initializing variables, classes, objects and imports) when we run a program.
Assuming we're writing a React program with files named `A.jsx`, `B.jsx` and `C.jsx` with the following codes in them

`A.jsx`

```jsx
import B from './B';
import C from './C';

function A() {
  return (
    <div>
      <C section={'About: My name is John Doe...'} />
      <C section={'Work: I work at ...'} />
      <C section={'Education: I went to ...'} />

      <B text={'toggle dark mode'} />
    </div>
  );
}

export default A;
```

`B.jsx`

```jsx
function B(props) {
  return <button>{props.text}</button>;
}

export default B;
```

`C.jsx`

```jsx
import B from './B';
function C(props) {
  return (
    <div>
      <p>{props.section}</p>
      <B text={'Find out More'} />
    </div>
  );
}

export default C;
```

From the above code example we can see;

- `A.jsx` imports (depends on) `B.jsx` and `C.jsx`
- `B.jsx` has no imports (0 dependencies)
- `C.jsx` imporst `B.jsx`

Diagram to represent the above structure
![Dependency structure of files](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5p2oiiucyypk9ei8h70g.png)
When building the above program, the order of the files in your working directory doesn't matter. The files are built in such a way that before a file is built all of it's dependencies (imports) musts be built first, which means the first file to be built must not import any file. Sorting the files in such a way is called **topological sort** which then gives the order in which the files would be built (**build order**). The build order for the above program will be.
![build order of a program](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yy2ldfvi6qa2ckis2p4v.png)

####Cyclic dependencies
Let's modify the our previous code examples

`A.jsx`

```jsx
import B from './B';
import C from './C';

function A() {
  return (
    <div>
      <C section={'About: My name is John Doe...'} />
      <C section={'Work: I work at ...'} />
      <C section={'Education: I went to ...'} />

      <B text={'toggle dark mode'} />
    </div>
  );
}

export default A;
```

`B.jsx`

```jsx
import { D } from './C';

function B(props) {
  return (
    <div>
      <button>{props.text}</button>
      <D />
    </div>
  );
}

export default B;
```

`C.jsx`

```jsx
import React from 'react';
import B from './B';

export default function C(props) {
  return (
    <div>
      <p>{props.section}</p>
      <B text={'Find out More'} />
    </div>
  );
}

export function D() {
  return <span>more info</span>;
}
```

From the above second code example `B.jsx` now imports a component from `C.jsx`
Dependency structure of second code example
![Cyclic dependency struct representation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xtot9fx7nml1xyns5j57.png)

The diagram shows that `B.jsx` and `C.jsx` have a circular dependency, as `B.jsx` imports `C.jsx` and `C.jsx` imports `B.jsx`. This creates a catch-22 situation because both files need to be built in order for the other to be built. In other words, `B.jsx` and `C.jsx` **depend on each other in a circular manner**.

This is what the cyclic dependency errors are all about, to avoid this errors for this example, you'll move `D` component into a separate file and import it into B from that new file that way `B.jsx` no longer depends on `C.jsx`
Example
`B.jsx`

```jsx
import D from './D';

function B(props) {
  return (
    <div>
      <button>{props.text}</button>
      <D />
    </div>
  );
}

export default B;
```

`D.jsx`

```jsx
export default function D() {
  return <span>more info</span>;
}
```

Our new file dependency structure will now look like this
![Fixed cicular dependency](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rx52xst3gqgac1wngo0l.png)
Alternatively you can move `D` component into `B.jsx`

##Topological Sort
In this discussion, we have covered the concept of cyclic dependency errors, how to resolve them, and the build order of files in a program. Now, we will delve into one of the algorithms that is used to determine the build order: **topological sorting**.
To use topological sorting to create a build order, the dependencies between the files must be represented as a directed acyclic graph (DAG). The algorithm then processes the nodes of the graph in a specific order, such that for every edge (A, B => B imports A) from node A to node B, node A appears before node B in the ordering.

Making a map from the previous files example.
`A.jsx` => [`B.jsx`, `C.jsx`]
`B.jsx` => [`D.jsx`]
`C.jsx` => [`B.jsx`]
`D.jsx` => []

In graph theory, dependencies are also known as indegrees. In graph theory, an indegree of a vertex in a directed graph is the number of edges that have that vertex as their destination (number of imports a files has).
`A.jsx` has an indigree of 2
`B.jsx` has an indigree of 1
`C.jsx` has an indigree of 1
`D.jsx` has an indigree of 0

In a topological sort we start with a file that has an indigree of 0 and then we subtract one from the indigrees of all the files that import that file, then we pick another file with indigree of 0 and we repeat the process until we reach the end
So we start with `D.jsx` then we subtract one from the indigree of `B.jsx` after subtracting one `B.jsx` will have an indigree of 0 so we continue with `B.jsx`. `C.jsx` imports `B.jsx` we subtract one from the indigree of `C.jsx`, `A.jsx` also imports `B.jsx` so it's indigree is also subtracted by one. The indigree of `C.jsx` will become 0 and that of `A.jsx` will be 1 so we pick `C.jsx` and since `A.jsx` imports `C.jsx` we subtract one from it's indigree and `A.jsx` will have an indigree of 0.

The final build order is:
`D.jsx` -> `B.jsx` -> `C.jsx` -> `A.jsx`

A Go implementation of topological sort

```go
package main

import "fmt"

func topologicalSort(graph map[string][]string) []string {
	// File and it's indigree
	indigrees := map[string]int{}
	// Make sure we don't add the same file twice
	visited := map[string]bool{}

	// The Final build order
	buildOrder := []string{}

	// Queue to add files with indigree of 0
	q := []string{}

	// Calculate indigrees
	for file, imports := range graph {
		indigree := len(imports)
		indigrees[file] = len(imports)
		if indigree == 0 {
			q = append(q, file)
		}
	}

	for len(q) > 0 {
		current := q[0]
		q = q[1:]
		buildOrder = append(buildOrder, current)
		visited[current] = true
        // Subtract one from all files that import current
		for file, imports := range graph {
			if !visited[file] {
				for _, val := range imports {
					if val == current {
						indigrees[file]--
						if indigrees[file] == 0 {
							q = append(q, file)
						}
						break
					}
				}
			}
		}
	}
	return buildOrder
}

func main() {

	// Files processed into a map
	graph := map[string][]string{
		"A.jsx": {"B.jsx", "C.jsx"},
		"B.jsx": {"D.jsx"},
		"C.jsx": {"B.jsx"},
		"D.jsx": {},
	}

	buildOrder := topologicalSort(graph)
	fmt.Println(buildOrder) // => [D.jsx B.jsx C.jsx A.jsx]
}
```

##Conclusion
Hope you've learned something, try implementing topological sort in javaScript. If you have any questions ask them in the comments and I'll answer them.
