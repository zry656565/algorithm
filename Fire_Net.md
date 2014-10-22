#Fire Net

This problem came frome ZOJ: [Problem 1002](http://acm.zju.edu.cn/onlinejudge/showProblem.do?problemCode=1002)

##Problem:

Suppose that we have a square city with straight streets. A map of a city is a square board with n rows and n columns, each representing a street or a piece of wall.

A blockhouse is a small castle that has four openings through which to shoot. The four openings are facing North, East, South, and West, respectively. There will be one machine gun shooting through each opening.

Here we assume that a bullet is so powerful that it can run across any distance and destroy a blockhouse on its way. On the other hand, a wall is so strongly built that can stop the bullets.

The goal is to place as many blockhouses in a city as possible so that no two can destroy each other. A configuration of blockhouses is legal provided that no two blockhouses are on the same horizontal row or vertical column in a map unless there is at least one wall separating them. In this problem we will consider small square cities (at most 4x4) that contain walls through which bullets cannot run through.

The following image shows five pictures of the same board. The first picture is the empty board, the second and third pictures show legal configurations, and the fourth and fifth pictures show illegal configurations. For this board, the maximum number of blockhouses in a legal configuration is 5; the second picture shows one way to do it, but there are several other ways.

![Fire Net][1]

Your task is to write a program that, given a description of a map, calculates the maximum number of blockhouses that can be placed in the city in a legal configuration.

The input file contains one or more map descriptions, followed by a line containing the number 0 that signals the end of the file. Each map description begins with a line containing a positive integer n that is the size of the city; n will be at most 4. The next n lines each describe one row of the map, with a '.' indicating an open space and an uppercase 'X' indicating a wall. There are no spaces in the input file.

For each test case, output one line containing the maximum number of blockhouses that can be placed in the city in a legal configuration.

###Sample input:

```
4
.X..
....
XX..
....
2
XX
.X
3
.X.
X.X
.X.
3
...
.XX
.XX
4
....
....
....
....
0
```

###Sample output:

```
5
1
5
2
4
```

##Solution 1 (Non-recursive) - Programming Language: C

- Run Time: ~0s
- Run Memory: 168kb

```c
#include <stdio.h>
#include <stdlib.h>

#define isNULL(point) ((point).x == -1)
#define NullP ((Point){-1, -1})

enum type { Blank, Wall, Blockhouse };
typedef struct _Point
{
    int x;
    int y;
} Point;
int placeable(int **grid, int size, Point p);
Point put_blockhouse(int **grid, int size, Point start, Point* stack, int* stackSize);
Point next_point(Point p, int size);
Point remove_blockhouse(int **grid, int size, Point* stack, int* stackSize);


int main(void) {
    int **grid;
    int size;
    int maxSize = 0;
    Point blockhouseStack[20];
    int blockhouseNum = 0;
    int first = 1;
    while (scanf("%d\n", &size) != EOF) {
        if (size <= 0) {
            break;
        }
        int i, j;
        /* initialize grid */
        grid = (int**) malloc(size * sizeof(int*));
        for (j = 0; j < size; j++) {
            grid[j] = (int*) malloc(size * sizeof(int));
        }

        /* store input to [grid] */
        char c;
        for (j = 0; j < size; j++) {
            for (i = 0; i < size; i++) {
                scanf("%c", &c);
                switch (c) {
                case 'X':
                    grid[j][i] = Wall;
                    break;
                case '.':
                    grid[j][i] = Blank;
                    break;
                }
            }
            /* pop \n */
            getchar();
        }

        /* put blockhouses */
        Point start = {0, 0};
        while (1) {
#ifdef debug
            printf("STACK:");
            for (int k = 0; k < blockhouseNum; k++) {
                printf("(%d, %d) ", blockhouseStack[k].x, blockhouseStack[k].y);
            }
            printf("\n");
#endif
            if (!isNULL(start = put_blockhouse(grid, size, start, blockhouseStack, &blockhouseNum))) {
                (maxSize < blockhouseNum) && (maxSize = blockhouseNum);
            } else {
                if (blockhouseNum == 0) {
                    break;
                }
                start = remove_blockhouse(grid, size, blockhouseStack, &blockhouseNum);
                start = next_point(start, size);
                if (isNULL(start) && blockhouseNum == 0) {
                    break;
                }
            }
        }

        /* do this to meet the format that ZOJ needs */
        if (!first) {
            printf("\n");
        } else {
            first = 0;
        }

        printf("%d", maxSize);

        /* free grid */
        for (j = 0; j < size; j++) {
            free(grid[j]);
        }
        free(grid);
        blockhouseNum = 0;
        maxSize = 0;
    }
    return 0;
}

int placeable(int **grid, int size, Point p) {
    int x = p.x, y = p.y;
    if (grid[y][x] != Blank) {
        return 0;
    }
    /*up*/
    while (--y >= 0) {
        if (grid[y][x] == Blockhouse) {
            return 0;
        } else if (grid[y][x] == Wall) {
            break;
        }
    }
    /*down*/
    y = p.y;
    while (++y < size) {
        if (grid[y][x] == Blockhouse) {
            return 0;
        } else if (grid[y][x] == Wall) {
            break;
        }
    }
    /*left*/
    y = p.y;
    while (--x >= 0) {
        if (grid[y][x] == Blockhouse) {
            return 0;
        } else if (grid[y][x] == Wall) {
            break;
        }
    }
    /*right*/
    x = p.x;
    while (++x < size) {
        if (grid[y][x] == Blockhouse) {
            return 0;
        } else if (grid[y][x] == Wall) {
            break;
        }
    }
    /*pass all test*/
    return 1;
}
Point put_blockhouse(int **grid, int size, Point start, Point* stack, int* stackSize) {
    if (isNULL(start)) {
        return NullP;
    }
    int i = start.x, j = start.y;
    for (; j < size; j++) {
        for (; i < size; i++) {
            if (placeable(grid, size, (Point){i, j})) {
                grid[j][i] = Blockhouse;
                stack[(*stackSize)++] = (Point){i, j};
                return (Point){0, 0};
            }
        }
        i = 0;
    }
    return NullP;
}
Point next_point(Point p, int size) {
    if (isNULL(p)) {
        return NullP;
    }
    if (p.x + 1 < size) {
        p.x += 1;
    } else {
        if (p.y + 1 < size) {
            p.y += 1;
            p.x = 0;
        }
        else {
            return NullP;
        }
    }
    return p;
}
Point remove_blockhouse(int **grid, int size, Point* stack, int* stackSize) {
    if (stackSize > 0) {
        Point p = stack[--(*stackSize)];
        grid[p.y][p.x] = Blank;
        return p;
    } else {
        return NullP;
    }
}
```

[1]: /images/firenet.gif
