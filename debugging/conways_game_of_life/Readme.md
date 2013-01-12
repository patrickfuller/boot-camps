In this example, we will bring together everything we learned about debugging to fix a script that runs Conway's game of life. Before we do that, however, let's answer the most obvious question here...

##What is Conway's game of life?

Basically, it's a discrete two-dimensional grid where every cell can be "alive" or "dead". Starting with a set of cells marked "alive", we can "evolve" the cells to the next generation by following these simple rules:

 * Any living cell with fewer than two neighbors dies (underpopulation)
 * Any living cell with more than three neighbors dies (overpopulation)
 * Any empty cell with three neighbors becomes a living cell (reproduction)   

There are a lot of cool things you can do with this, and, for that, I defer to the [wiki](http://en.wikipedia.org/wiki/Conway's_Game_of_Life). Let's get on with debugging.

The code below is intended to be an implementation of Conway's game of life, but it's pretty terrible.

```python
from math import sqrt

def conway(population, 
    generations = 100):
    for i in range(genrations): population = evolve(population)
    return popluation

def evolve(population):
    activeCells = population[:]
    for cell in population:
        for neighbor in neighbors(cell):
            if neighbor not in activeCells: activeCells.append(neighbor)
    newPopulation = []
    for cell in activeCells:
        count = sum(neighbor in population for neighbor in neighbors(cell))
        if count == 3 or (count == 2 and cell in population): 
            if cell not in newPopulation: newPopluation.add(cell)
    return newPopulation

def neighbors(cell):
    x, y = cell
    return [(x, y), (x+1, y), (x-1, y), (x, y+1), (x, y-1), (x+1, y+1), (x+1, y-1), (x-1, y+1), (x-1, y-1)]

glider = [(0, 0), (1, 0), (2, 0), (0, 1), (1, 2)]
print conway(glider)
```

##Step 1: Linting

Let's start by running it through `pyflakes` and fixing the blatant typos. Running "`pyflakes 1_conway_pre_linted.py`" gives:

```
1_conway_pre_linted.py:9: 'sqrt' imported but unused
1_conway_pre_linted.py:13: undefined name 'genrations'
1_conway_pre_linted.py:14: undefined name 'popluation'
1_conway_pre_linted.py:25: undefined name 'newPopluation'
```

It caught an unused import and some typos. In this little example, `pyflakes` saved us from having to run the script multiple times to find each error; imagine how useful that is in larger code! If we fix the code, it should now run.

```python
def conway(population, 
    generations = 100):
    for i in range(generations): population = evolve(population)
    return population

def evolve(population):
    activeCells = population[:]
    for cell in population:
        for neighbor in neighbors(cell):
            if neighbor not in activeCells: activeCells.append(neighbor)
    newPopulation = []
    for cell in activeCells:
        count = sum(neighbor in population for neighbor in neighbors(cell))
        if count == 3 or (count == 2 and cell in population):
            if cell not in newPopulation: newPopulation.append(cell)
    return newPopulation

def neighbors(cell):
    x, y = cell
    return [(x, y), (x+1, y), (x-1, y), (x, y+1), (x, y-1), (x+1, y+1), (x+1, y-1), (x-1, y+1), (x-1, y-1)]

glider = [(0, 0), (1, 0), (2, 0), (0, 1), (1, 2)]
print conway(glider)
```

Running this gives

```
[(0, 0), (2, 0), (1, 2), (1, -1), (2, 1)]
```

##Step 2: Formatting

Now that we have running code, let's clean it up. Running "`pep8 2_conway_pre_formatted.py`",

```
2_conway_pre_formatted.py:10:23: W291 trailing whitespace
2_conway_pre_formatted.py:11:5: E128 continuation line under-indented for visual indent
2_conway_pre_formatted.py:11:5: E125 continuation line does not distinguish itself from next logical line
2_conway_pre_formatted.py:11:16: E251 no spaces around keyword / parameter equals
2_conway_pre_formatted.py:11:18: E251 no spaces around keyword / parameter equals
2_conway_pre_formatted.py:12:32: E701 multiple statements on one line (colon)
2_conway_pre_formatted.py:15:1: E302 expected 2 blank lines, found 1
2_conway_pre_formatted.py:19:43: E701 multiple statements on one line (colon)
2_conway_pre_formatted.py:24:41: E701 multiple statements on one line (colon)
2_conway_pre_formatted.py:27:1: E302 expected 2 blank lines, found 1
2_conway_pre_formatted.py:29:80: E501 line too long (107 > 79 characters)
2_conway_pre_formatted.py:29:23: E225 missing whitespace around operator (x12)
```

Wow, that's a lot of issues! I recommend going through each one and figuring out why it deviates from the PEP8 standard. It's obnoxious, but it'll help you write better code the first time around. 

Even with all of these issues, there are still some things that `pep8` isn't telling us.

 * Most importantly, where are the comments!? _Always_ comment code!
 * Some variables are in `camelCase`. That's bad. Rewrite as `snake_case`.
 * Never compress `if` statements to one line. Don't be afraid of whitespace.

Fixing all of these issues results in much more readable code.

```python
def conway(population, generations=100):
    """Runs Conway's game of life on an initial population."""
    for i in range(generations):
        population = evolve(population)
    return population


def evolve(population):
    """Evolves the population by one generation."""
    # Get a unique set of discrete cells that need to be checked
    active_cells = population[:]
    for cell in population:
        for neighbor in neighbors(cell):
            if neighbor not in active_cells:
                active_cells.append(neighbor)
    # For each cell in the set, test if it lives or dies
    new_population = []
    for cell in active_cells:
        count = sum(neighbor in population for neighbor in neighbors(cell))
        if count == 3 or (count == 2 and cell in population):
            if cell not in new_population:
                new_population.append(cell)
    # Return the new surviving population
    return new_population


def neighbors(cell):
    x, y = cell
    return [(x, y), (x + 1, y), (x - 1, y), (x, y + 1), (x, y - 1),
            (x + 1, y + 1), (x + 1, y - 1), (x - 1, y + 1), (x - 1, y - 1)]

glider = [(0, 0), (1, 0), (2, 0), (0, 1), (1, 2)]
print conway(glider)
```

##Step 3: Deep-rooted debugging

According to successful implementations of Conway's game of life, our given initial population should cause the living cells to "glide" across the grid over many generations. However, it's not doing that. What's going on?

Using `pdb`, we can follow the state of every variable as the code runs. This is difficult to show through IPython, so I'll leave it up to you. Set some traces in "`3_conway_pre_debugged.py`" and see if you can find the bug. Once you find it (or give up), keep reading.

######**SPOILER:** The issue is that I accidentally defined `(x, y)` as a neighbor. Removing that fixes the code, and the glider glides.

##Step 4: Speed and profiling

This code is now functional and follows PEP8 standards, but maybe it could be faster. For a baseline, let's time the current script over 1,000,000 generations. Running "`time python 4_conway_pre_profiled.py`" results in

```
real	1m9.455s
user	1m9.228s
sys	 0m0.012s
```

To break down the locations of the speed constraints, we can run "`python -m cProfile -s time 4_conway_pre_profiled.py`"

```
   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
198000000   38.248    0.000   38.248    0.000 4_conway_pre_profiled.py:28(<genexpr>)
  1000000   28.373    0.000  100.867    0.000 4_conway_pre_profiled.py:17(evolve)
 22000000   17.818    0.000   56.066    0.000 {sum}
 27000000   15.030    0.000   15.030    0.000 4_conway_pre_profiled.py:36(neighbors)
 22000000    1.398    0.000    1.398    0.000 {method 'append' of 'list' objects}
        1    0.616    0.616  101.496  101.496 4_conway_pre_profiled.py:10(conway)
        1    0.013    0.013    0.013    0.013 {range}
        1    0.000    0.000  101.496  101.496 4_conway_pre_profiled.py:7(<module>)
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
```

Going through everything here is _waay_ beyond the scope of the tutorial. However, I want to introduce one speed boost that will not only improve performance, but will also simplify our code and improve its readability. This involves the use of _sets_, which are collections that only store unique elements. They are much better than lists when you need to check for the existence of an element, which we do a lot in this script. Refactoring to use sets results in:

```python
def conway(population, generations=100):
    """Runs Conway's game of life on an initial population."""
    population = set(population)
    for i in range(generations):
        population = evolve(population)
    return list(population) 


def evolve(population):
    """Evolves the population by one generation."""
    # Get a unique set of discrete cells that need to be checked
    active_cells = population | set(neighbor for p in population
                                    for neighbor in neighbors(p))
    # For each cell in the set, test if it lives or dies
    new_population = set()
    for cell in active_cells:
        count = sum(neighbor in population for neighbor in neighbors(cell))
        if count == 3 or (count == 2 and cell in population):
            new_population.add(cell)
    # Return the new surviving population
    return new_population


def neighbors(cell):
    """Returns the neighbors of a given cell."""
    x, y = cell
    return [(x + 1, y), (x - 1, y), (x, y + 1), (x, y - 1), (x + 1, y + 1),
            (x + 1, y - 1), (x - 1, y + 1), (x - 1, y - 1)]

glider = [(0, 0), (1, 0), (2, 0), (0, 1), (1, 2)]
print conway(glider)
```

The code now clocks in at 50.3 seconds, which is 38% increase in speed. That's nice.

##Closing thoughts

This notebook just showcased linting, coding styles, finding bugs, and improving code speed. I know this all seems like a lot of work, but it will make you produce much better code (and, as you get better, your debugging time will decrease rapidly). Stick with it!
