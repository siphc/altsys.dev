+++
title = "What The Heck is an Arena?"
date = "2025-12-07"
author = "William Pan"
tags = ["c", "memory-management"]
description = "Imagine this scenario. You are developing an AST builder for your new shiny compiler; at some point in the program, you come up with this schema for dealing with floating point arithmetic..."
+++

### Introduction

Imagine this scenario. You are developing an AST builder for your new shiny compiler; at some point in the program, you come up with this schema for dealing with floating point arithmetic:

```c
typedef enum {
    NODE_NUMBER,
    NODE_VAR,
    NODE_BINOP,
} NodeType;

typedef struct Node {
    NodeType type;
    union {
        double number;              // NODE_NUMBER
        const char *name;           // NODE_VAR
        struct {                    // NODE_BINOP
            char op;
            struct Node *left;
            struct Node *right;
        } binop;
    };
} Node;
```

Because you haven't heard of arenas yet, you go with the obvious approach, which inevitably leaves your code looking something like this:

```c
static Node *make_number(double num) {
    Node *n = malloc(sizeof(Node));	// OK, looks alright.
    n->type = NODE_NUMBER;
    n->number = num;
    return n;
}

static Node *make_var(const char *name, size_t len) {
    Node *n = malloc(sizeof(Node));	// malloc() once for the node...
    n->type = NODE_VAR;
    n->name = malloc(len + 1);		// and twice for the actual variable.
    memcpy(n->name, name, len);
    n->name[len] = '\0';
    return n;
}
```

(We'll ignore the parser of the program, since it's quite verbose and doesn't actually do anything interesting.)

---

### The Problem

Have you noticed a problem with this code?

How the heck do you free everything that you allocated?

Being the memory wizard that you are, you decide to traverse postorder the entire AST, freeing every node as you go. No biggie.

But what if one of your `malloc()`s failed during the midst of construction? What if you ran out of memory? What if you add more node types -- say, unary operators? You'd have to make amendments to your freeing function. Ternary operators? You'd have to make another amendment. Every `malloc()` call also comes up with a header -- for your tiny little NODE_NUMBER, you'd have (enum + padding + op + padding + two pointers) = 32 bytes for the node itself, and likely half of that additionally for the header. You end up littering all over your heap, which end up to also be terrible news for your cache, as well.

This is just painful. You decide to read more about memory allocation in C. Then you came across arenas.

---

### The Solution

The concept of arenas is simple. In a program like this, many allocations share the same lifetime, so it doesn't really make sense for you to individually allocate every single one of them, only to free them all at the same time. An arena is just some chunk of memory that you use to store all your data that shares the same lifetime.

What does that look like in practice? Let's try rewriting your AST builder with an arena:

```c
typedef struct {
    uint8_t *buffer;	// The actual memory itself
    size_t capacity;
    size_t offset;		// Basically, "how much did we already 'allocate'?"
} Arena;

Arena arena_create(size_t cap) {
    Arena a = { malloc(cap), cap, 0 };
    return a;
}

void *arena_alloc(Arena *a, size_t size) {
    size_t aligned = (a->offset + 7) & ~7;	// Bit hacking to align to 8 bytes
    if (aligned + size > a->capacity) {
        fprintf(stderr, "Arena OOM\n");
        exit(1);
    }
    void *p = a->buffer + aligned;
    a->offset = aligned + size;
    return p;
}

void arena_destroy(Arena *a) {
    free(a->buffer);
}
```

The magic here lies in the line `void *p = a->buffer + aligned;`. We calculate and return some chunk of aligned, allocated memory that the user asks for, without ever doing any manual allocation, because we've already done that! **One** `malloc()` is all you need.

You don't need to even make substantial changes to the rest of your code; it's all analogous.

```diff
static Node *make_var(const char *name, size_t len) {
-   Node *n = malloc(sizeof(Node));	// malloc() once for the node...
+	Node *n = arena_alloc(p->arena, sizeof(Node));

   	n->type = NODE_VAR;

-   n->name = malloc(len + 1);		// and twice for the actual variable.
+	char *var = arena_alloc(p->arena, len + 1);
-   memcpy(n->name, name, len);
+	memcpy(var, name, len);
-   n->name[len] = '\0';
+	var[len] = '\0';

+	n->name = var;
    return n;
}
```

---

### Finishing Thoughts

Free yourself from `malloc()` hell. If for some reason you find yourself needing to do a lot of manual allocations, ... you don't! *Every* object that shares the same lifetime can be organized into a single arena -- then you can just allocate and deallocate only once.

For a little extra content, I benchmarked both approaches. Note the amount of allocs and frees: the manual approach has 45 of each, while the arena approach only has 2 (one for the actual input).

```bash
[svi@nixos:~/temp]$ cc -o arena_ast arena_ast.c
[svi@nixos:~/temp]$ cc -o manual_ast manual_ast.c
[svi@nixos:~/temp]$ valgrind ./arena_ast
==2270451== Memcheck, a memory error detector
==2270451== Copyright (C) 2002-2024, and GNU GPL'd, by Julian Seward et al.
==2270451== Using Valgrind-3.26.0 and LibVEX; rerun with -h for copyright info
==2270451== Command: ./arena_ast
==2270451==
Input:  a + b + c + d + e + a + b + c + d + e + a + b + c + d + e
Result: 15.00  (assuming var=1)
Arena:  1048 bytes used
==2270451==
==2270451== HEAP SUMMARY:
==2270451==     in use at exit: 0 bytes in 0 blocks
==2270451==   total heap usage: 2 allocs, 2 frees, 5,120 bytes allocated
==2270451==
==2270451== All heap blocks were freed -- no leaks are possible
==2270451==
==2270451== For lists of detected and suppressed errors, rerun with: -s
==2270451== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)

[svi@nixos:~/temp]$ valgrind ./manual_ast
==2271111== Memcheck, a memory error detector
==2271111== Copyright (C) 2002-2024, and GNU GPL'd, by Julian Seward et al.
==2271111== Using Valgrind-3.26.0 and LibVEX; rerun with -h for copyright info
==2271111== Command: ./manual_ast
==2271111==
Input:  a + b + c + d + e + a + b + c + d + e + a + b + c + d + e
Result: 15.00
==2271111==
==2271111== HEAP SUMMARY:
==2271111==     in use at exit: 0 bytes in 0 blocks
==2271111==   total heap usage: 45 allocs, 45 frees, 1,982 bytes allocated
==2271111==
==2271111== All heap blocks were freed -- no leaks are possible
==2271111==
==2271111== For lists of detected and suppressed errors, rerun with: -s
==2271111== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```

Have fun!