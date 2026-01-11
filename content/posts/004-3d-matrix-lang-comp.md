+++
title = "A Comparison of Dynamic 3d Array Instantiation in Selected Languages"
date = "2026-01-10"
author = "siphc"
tags = ["pl", "random"]
description = "Sometimes I find myself manually typing the solution in C++ and wondering if the grass is greener on the other side."
+++

### Python
```python
matrix = [[[0 for _ in range(l)] for _ in range(m)] for _ in range(n)]
```

### Java
```java
int[][][] matrix = new int[n][m][l];
```

### Haskell
```haskell
matrix = replicate n (replicate m (replicate l 0))
```

### OCaml
```ocaml
let matrix = Array.init n (fun _ -> Array.init m (fun _ -> Array.make l 0))
```

### C++11
```cpp
#include <vector>

std::vector<std::vector<std::vector<int>>> matrix(n,
	std::vector<std::vector<int>>(m,
		std::vector<int>(l)));
```

### C99 (VLA)
```c
#include <stdlib.h>

int (*matrix)[m][l] = malloc(sizeof(int[n][m][l]));
```

### C89
```c
#include <stdlib.h>

int ***matrix;
int i, j;

matrix = malloc(n * sizeof(int **));
for (i = 0; i < n; i++) {
    matrix[i] = malloc(m * sizeof(int *));
    for (j = 0; j < m; j++) {
        matrix[i][j] = calloc(l, sizeof(int));
    }
}
```