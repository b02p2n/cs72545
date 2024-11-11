java c
MACS 30121 - Computer Science with Social Science Applications I documentation
Modeling Language Shifts
The goal of this assignment is to give you practice using nested loops, lists of lists, and functions. The algorithms you need to implement for this assignment are more challenging than the fi rst assignment. We recommend starting early.
Getting started
Please accept the invitation for this assignment using the URL on: Canvas-->Assignments-->Programming Assignments-->PA2.
You will fi nd the fi les you need for the programming assignment directly in the root of your repository, including a README.txt fi le that explains what each fi le is. Make sure you read that fi le.
Next, make sure to review the Coursework Basics (../../resources/coursework-basics.html#coursework-basics) page. As described in that page, we strongly encourage you to start an IPython session to experiment with your code. Again, the steps are similar to those for the short exercises: open up a new terminal window and navigate to your pa2-$GITHUB_USERNAME directory. Then, fi re up ipython3 from the Linux command-line, set up autoreload, and import your code as follows:
$ ipython3
In [1]: %load_ext autoreload
In [2]: %autoreload 2
In [3]: import language
Finally, for this assignment, you may assume that the input passed to your functions has the correct format. You may not alter any of the input that is passed to your functions. In general, it is bad style. to modify a data structure passed as input to a function, unless that is the explicit purpose of the function. The user or client of your function may have other uses for the data structure and should not be surprised by unexpected changes.
Introduction
Many regions around the world support populations that speak more than one language. For example, both Spanish and Catalan are spoken in the Valencia region of Spain (strictly speaking, the language spoken in Valencia is a form. of Catalan known as Valencian). Catalan is more notably spoken in the Catalonia region, but we will be focusing on the use of Catalan in the Valencia region. More precisely, some of the population of Valencia is monolingual (only speaks Spanish) and the rest are bilingual (speak Spanish and Catalan). Elsewhere along the Mediterranean coast, populations are French/French-and-Catalan and Italian/Italian-and-Catalan speakers.
When there are two languages spoken by a population, a hierarchy is often established. One language becomes the dominant language (DL) and is spoken by everyone in the population. The other is the subordinate language (SL), which is spoken by only some of the population. In Valencia, Spanish is the dominant language and Catalan is the subordinate language.
It is possible for both languages to survive side-by-side, but often the SL dies out in favor of the DL. The process of speakers abandoning the SL to speak the DL is called a language shift, which is complete when there are no living speakers of the SL.
Language preferences are transmitted from parents to their children. Parents decide which language habits to pass to their children by considering their engagement with language. For example, if a SL speaker is highly engaged with the SL (perhaps there are many opportunities to speak the SL in their community), they will speak to their children in the SL and the SL will be passed to a new generation. On the other hand, if an SL speaker is weakly engaged with the SL, they might abandon the SL and speak to their children in the DL.
Your task in this assignment is to implement a variation of the language shift model described by Beltran, et al. in their paper A Language Shift Simulation Based on Cellular Automata (https://www.researchgate.net/publication/259557981_A_Language_Shift_Simulation_Based_on_Cellular_Automata). You will start with a region of DL and SL speakers and simulate how language preferences change over time. You will especially consider the conditions that facilitate language shifts. Please note that you do not need to read the Beltran paper to do this programming assignment.
Before you start your implementation, we will specify the details of the components of the abstract model you will be using, and then describe the concrete data structures you will be using to implement the model.
Our model
For the model, we need to specify the:
a region,
the speakers in a region,
a speaker’s neighborhood,
community centers in a region,
a speaker’s engagement level,
language transmission rules,
a step in the simulation, and
the stopping conditions for the simulation.
Regions
A region is modeled as a N by N grid, where the value of N can be diff erent for diff erent regions. Each grid location (sometimes called a cell) represents a home in the region that houses speakers with a language preference.
Here is a sample region with N = 5. This fi gure shows the location of each home (cell) in the region.
Sample region

Speakers
Each cell in the grid is home to some speakers that have a language preference. Language preference is one of three diff erent states and indicates the level of engagement with the SL:
State 0: Speakers are monolingual and do not speak the SL.
State 1: Speakers are bilingual (they speak both the SL and the DL), but only speak the SL in certain situations.
State 2: Speakers are bilingual and prefer to speak the SL whenever possible.
Here is a sample region with speakers that we will use throughout this write-up. A white cell indicates that a home is occupied by speakers in language state 0, a light grey cell indicates that a home is occupied by speakers in language state 1, and a dark grey cell indicates that a home is occupied by speakers in language state 2.
Sample region with speakers

Neighborhood
The neighborhood of a home will be defi ned by a parameter R. The R-neighborhood of a home at location (i, j) in an N by N region contains all locations (k, l) such that:
0 <= k < N
0 <= l < N
|k - i| <= R
|l - j| <= R
This is often called a Moore neighborhood. A neighborhood with parameter R = x is called an “R-x neighborhood”.
The following fi gures show the neighborhoods around locations (2, 2) and (3, 0) for diff erent values of R. We use yellow to indicate that a location is included in the specifi ed neighborhood and white to indicate that a location is not included in the neighborhood.
Neighborhood around (2, 2)                                                          Neighborhood around (3, 0)
R = 0                                         R = 1                                         R = 1                                       R = 2

Notice that location (3, 0), which is closer to the boundaries of the region, has fewer neighbors for the same value of R than location (2, 2), which is the middle of the region. Also notice that location (i, j) is considered part of its own neighborhood.
Community centers
Each region also contains some number of community centers. Community centers off er SL-speaking homes within a service distance D opportunities to engage with the SL. A community center at location (i, j) with service distance D provides services for SL-speaking homes in the R-D neighborhood around (i, j). That is, a community center services homes within its Moore neighborhood with parameter R = D. Community centers do not serve DL-speaking homes.
Sample region with a community center
Community center location: (1, 2)
Community center service distance: 1


For example, say our sample region has a community center at location (1, 2) with service distance 1. The community center is shown above with a bold outline and its R-1 neighborhood is shaded in yellow. The SL-speaking homes at locations (0, 2), (0, 3), (1, 1), (1, 2), (1, 3), (2, 2) and (2, 3) are within the service distance of the community center, while those farther away are not. Again, community centers do not service DL-speaking homes.
Notice that homes and community centers can overlap. In this example, there is a home at location (1, 2) with speakers in language state 1 and a community center at location (1, 2). You can think of this as there being an apartment above a store.
Although our sample region has just one, a region can have any number of community centers.
Engagement level
We defi ne an engagement level for a home at a specifi ed location to be E = S/T . S is the sum of the language preferences (states 0, 1, or 2) of the homes in the neighborhood centered at that location and T is the total number of homes in the neighborhood. You can think of this number as the opportunity for speakers to engage in the SL.
A home’s engagement level will always be a number between 0 and 2, inclusive. If the engagement level of a home is 0, everyone in their neighborhood (including themselves) has language state 0. Similarly, if the engagement level of a home is 2, everyone in their neighborhood has language state 2.
The fi gures below show the engagement level for each home in the sample region from before with R-1 neighborhoods and R-2 neighborhoods. The engagement level is shown beneath the location in each cell.
Engagement levels
Parameters:
R: 1
Sample region                                                                         Engagement levels


Parameters:
R: 2
Sample region                                                                         Engagement levels


Think carefully about how you will compute the engagement level of a home. You do not need to use an additional data structure (e.g., you don’t need to construct a list of locations in the neighborhood of a home to compute its engagement level). Nor do you need to check every location in the region.
Transmission rules
You will use the following rules to determine the language preferences of the next generation.
Parents pass their language preferences to their children according to their engagement level E and the following table:
Parent’s state:                                       Child’s state:
0                                 1                                 2
0                                                            E <= B                        B < E
1                                                            E < B                          B <= E <= C                C < E
2                                                            E <= A                        A < E < B                     B <= E
The values of A, B, C, which are thresholds that represent the engagement level needed to transition from one language state to another, are parameters of the simulation. Notice that it is not possible to transition from state 0 to state 2 in a single generation.
Furthermore, if an SL-speaking home (a home with speakers in state 1 or state 2) is within the service distance of a community center, children cannot inherit a lower language preference than their parents. You can think of the community center as providing enough engagement with the SL for a parent to pass down at least their own preference state regardless of how unengaged their neighbors may be.
Let’s apply the transmission rules to our sample region with an R-1 neighborhood. First, we will apply the transmission rules to locations in a region without any community centers.
Transmission example in a region without community centers
Parameters:
R: 1
Transition thresholds (A, B, C): (0.6, 0.8, 1.6)
Sample region                                                                         Engagement levels


Example 1: (0, 0): The speakers at location (0, 0) are currently in language state 0. Based on the current values of the neighbors of (0, 0) (shown in the image above), the speakers have a 0.25 engagement level in an R-1 neighborhood. Using the transmission table, the next language state of (0, 0) will be 0, since 0.25 <= B = 0.8.
Example 2: (1, 1): The speakers at location (1, 1) are currently in language state 1. Based on the current values of the neighbors of (1, 1) (again, shown in the image above), the speakers have an approximately 0.56 engagement level. Using the transmission table, the next language state of (1, 1) will be 0, since 0.56 < B = 0.8. This is an example of an SL-speaking home shifting to the DL.
Example 3: (1, 4): The speakers at location (1, 4) are currently in language state 0. Based on the current values of the neighbors of (1, 4), the speakers have a 1.0 engagement level. Using the transmission table, the next language state of (1, 4) will be 1, since B = 0.8 < 1.0. This is an example of a DL-speaking home picking up the SL.
Now let’s apply the transmission rules in a region with a community center at location (1, 2) with service distance 1. A bold outline denotes the location of the community center.
Transmission example in a region with a community center
Parameters:
R: 1
Transition thresholds (A, B, C): (0.6, 0.8, 1.6)
Community center location: (1, 2)
Community center service distance: 1
Sample region                                                                         Engagement levels


Example 4: (0, 0): The speakers at location (0, 0) are currently in language state 0. Based on the current values of the neighbors of (0, 0) (shown in the image above), the speakers have a 0.25 engagement level in an R-1 neighborhood. Using the transmission table, the next language state of (0, 0) will be 0, since 0.25 <= B = 0.8. Notice that this example is the same as Example 1.
Example 5: (1, 1): The speakers at location (1, 1) are currently in language state 1. Based on the current values of the neighbors of (1, 1), the speakers have an approximately 0.56 engagement level. Using the transmission table, the next language state of (1代 写MACS 30121 - Computer Science with Social Science Applications I documentationPython
代做程序编程语言, 1) should be 0, since 0.56 < B = 0.8. However, location (1, 1) is within the service distance of the community center at location (1, 2). Thus, the next language state of (1, 1) stays at 1.
Example 6: (3, 0): The speakers at location (3, 0) are currently in language state 1. Based on the current values of the neighbors of (3, 0), the speakers have an approximately 0.67 engagement level. Using the transmission table, the next language state of of (3, 0) will be 0, since 0.67 < B = 0.8. Notice that in this case, the location (3, 0) is not within the service distance of the community center at (1, 2).If the language state of an SL-speaking home were to decrease according to the transmission table, we check to see if they are close to a community center. If they are, their language state cannot go down. It’s not necessary to check whether an SL-speaking home whose language state will increase or stay the same is close to a community center. It’s also never necessary to determine whether or not a home in language state 0 is close to a community center, since community centers only serve SL-speaking homes.
Think carefully about how you will check whether a home is close to a community center. There is no need to visit all of the locations in the neighborhood of a community center in order to determine whether a home is within its service distance.
In the real-world, it’s common to build large programs in stages. Programmers start by implementing simplifi ed versions of their projects, then incrementally add more complex features. We recommend using this strategy here. Start by writing an initial implementation that computes the next language state for a home in a region without any community centers. Then, once you’re sure that works, add support for community centers.
Simulation step
A step in the simulation consists of fi nding the language preference for the next generation of all homes in a region. You will do this by going once over the grid and updating each home to refl ect the language preference of the next generation of speakers using the transmission rules described above.
More precisely, each step starts in the upper-left corner of the region (location (0, 0)) and moves through the region row by row until it reaches the lower-right corner (location (4, 4) in our sample city). The homes in a row will be visited one by one from left to right and the transmission rules are applied to a home at the time of the visit.
The fi gure below shows our sample region at the start of the step and the end of the step. A bold outline denotes the location of the community center.
Sample simulation step
Parameters:
R: 1
Transition thresholds (A, B, C): (0.6, 0.8, 1.6)
Community center location: (1, 2)
Community center service distance: 1
Region before the step                                              Region after the step


Stopping conditions
We will simulate steps until we have
1. executed a specifi ed maximum number of steps or
2. executed a step in which no homes have transitioned to a new language state.
Sample simulation
Parameters:
R: 1
Transition thresholds (A, B, C): (0.6, 0.8, 1.6)
Community center location: (1, 2)
Community center service distance: 1
Maximum number of steps: 5
Initial region                                                                           Final region


Data representation
In addition to having an abstract model of a region, we need to have concrete representations for speaker preferences, regions, locations, and community centers.
Representing speakers and language preferences
A home in a region will be represented by an integer indicating that home’s language preference. Language preference is one of three states (state 0, state 1, and state 2) described above. Thus, we will use an integer value 0, 1, or 2 to represent a home of speakers and their language preference.
Representing regions
We will represent a region using a list, where each element of the list is itself a list of integers, each representing a home. You may assume that these integers will always be one of 0, 1, or 2. We will refer to this data structure as the grid.
Here is the list representation of our sample region:
[[0, 0, 1, 1, 0],
[0, 1, 1, 2, 0],
[0, 0, 2, 2, 1],
[1, 2, 1, 1, 2],
[0, 1, 1, 0, 2]]
Representing locations
We will use tuples of two integers to represent locations. For example, the tuple (2, 4) represents the cell in row 2, column 4 of the grid.
Representing community centers
A community center will be represented by a tuple of two values. The fi rst value is the location of the community center (which is itself a tuple) and the second is the service distance of the community center. For example, our community center at location (1, 2) with service distance 1 is represented by the tuple ((1, 2), 1) .
All of the community centers in a region will be stored in a list. For example, if a region has two community centers, one at (1, 2) with service distance 1 and the other at (4, 3) with service distance 2, this will be represented by the list [((1, 2), 1), ((4, 3), 2)] .
If a region has no community centers, this is the empty list.
Task 1
Your goal in this task is to implement the function:
def run_simulation(grid, R, thresholds, centers, max_steps)
in the fi le language.py . This function executes the simulation described above on the specifi ed region grid until one of the stopping conditions has been met. This function must return a tuple of three integers representing the total number of homes in each language state at the end of the simulation. Furthermore, your function should modify the grid argument. By changing the grid in-place, the grid at the end of the simulation will refl ect the language preference changes that took place during the simulation.
You should think carefully about how to split this task into multiple functions. That is, you should write a series of auxiliary functions, each of which implements a subtask. When designing your function decomposition, keep in mind that you need to accomplish the following subtasks:
1. determine whether an SL-speaking home is within the service distance of a community center,
2. determine the engagement level of a home,
3. determine the language state of the next generation of speakers in a home according to the transmission rules,
4. simulate a step of the simulation, and
5. run the steps until one of the stopping conditions is met.
Do not combine the last two or three tasks into one mega-task. The resulting code would be hard to read and debug. Combining too many subtasks into a single function will have an impact on your code quality score. Don’t forget to include function headers comments (i.e., docstrings) in your helper functions!
Task 2
Beltran, et al. found that the value of the threshold parameter B has a large impact on whether a language shift occurs during a given simulation. They found that for small values of B, no language shift occurs and the DL and SL survive side by-side. However, when B is large, the SL dies out and all speakers move to language preference state 0 by the end of the simulation.
In this task, you will complete the function:
def simulation_sweep(grid, R, A, Bs, C, centers, max_steps)
in the fi le language.py , which takes one fl oat for the threshold A, one fl oat for the threshold C, and a list of fl oats for the threshold B. This function will run your simulation once for each of the values in the list Bs . It should return a list of frequency tuples, one for each run of the simulation. This type of experiment, where you run a simulation multiple times with diff erent values for one of the parameters, is often called a parameter sweep.
You should start with the same grid for each simulation, but remember that your run_simulation function modifi es the grid. It might be worthwhile to review the “Shallow copy versus deep copy” section of the “Lists, Tuples, and Strings” chapter of your textbook before beginning this part. You are welcome to make use of the the copy module in this task.
Do you expect to fi nd the same results as Beltran, et al.? Looking at the transmission table, these results make sense. If B is large, SL speakers need higher engagement levels for the next generation to have the same language preference state.
It is not feasible to learn the actual value of B in the real world, but practical steps can be taken to reduce the value of B. For example, initiatives like encouraging the use of the SL in communities or providing opportunities for DL speakers to learn the SL could have a real impact and help ensure that the SL survives.
Running your program
We strongly encourage you to run your functions as you write them! The surest way to guarantee that you will need to spend hours debugging is to write all your code and only then start testing.
You can fi nd sample regions to play with in the tests subdirectory of your pa2 directory. For example, the fi le tests/writeup-grid-with-cc.txt contains the sample region used throughout this writeup. The fi le tests/writeup-grid.txt contains the same region without the community center. Please see tests/README.txt for descriptions of each of the sample regions.
We have also provided a library called utilities.py that you might fi nd useful when you’re trying out your code. It contains the following functions for working with regions:
read_grid , which takes a string that is the name of a fi le containing a region and returns the corresponding region as a list of lists and a list of community centers in that region, as a tuple.
print_grid , which takes a region as a list of lists and prints the region, including N (the size of the region).
As you’re working on this assignment, we recommend trying out your functions in IPython. You might start to experiment like this:
In [1]: %load_ext autoreload
In [2]: %autoreload 2
In [3]: import language, utility
In [4]: region, centers = utility.read_grid("tests/writeup-grid-with-cc.txt")
In [5]: utility.print_grid(region)
N: 5
[0, 0, 1, 1, 0]
[0, 1, 1, 2, 0]
[0, 0, 2, 2, 1]
[1, 2, 1, 1, 2]
[0, 1, 1, 0, 2]
The session above:
sets IPython to automatically reload your code,
imports the modules (fi les that contain code) you’re working with (notice that you don’t need to include the .py extension),
reads a sample grid, and stores the region in a variable called region and a list of community centers in a variable called centers ,
and prints out the sample region.
From here, you can go on to test the region with some of your own functions.
Running run_simulation
We have included code in language.py that calls your run_simulation function and, if the region is small enough, prints the results.
Running language.py with the --help fl ag shows the fl ags to use for diff erent arguments:
$ python3 language.py --help
Usage: language.py [OPTIONS]
Run the simulation.
Options:
--grid_file PATH filename of the grid
--r INTEGER neighborhood radius
--a FLOAT transition threshold A
--b FLOAT transition threshold B
--c FLOAT transition threshold C
--max_steps INTEGER maximum number of simulation steps
--help Show this message and exit.
Here is a sample use of this program:
$ python3 language.py --grid_file tests/writeup-grid-with-cc.txt --max_steps 5
and here is the output it should print:
Running the simulation...
Initial region:
[0, 0, 1, 1, 0]
[0, 1, 1, 2, 0]
[0, 0, 2, 2, 1]
[1, 2, 1, 1, 2]
[0, 1, 1, 0, 2]
With community centers:
((1, 2), 1)
Final region:
[0, 0, 1, 1, 1]
[0, 1, 1, 2, 1]
[1, 1, 2, 2, 1]
[1, 2, 1, 1, 2]
[1, 1, 1, 1, 2]
Final language state frequencies: (3, 16, 6)
You can also specify the neighborhood radius R and any of the transition thresholds A, B, and C. For example:
$ python3 language.py --grid_file tests/writeup-grid-with-cc.txt --r 2 --b 1.2
will print:
Running the simulation...
Initial region:
[0, 0, 1, 1, 0]
[0, 1, 1, 2, 0]
[0, 0, 2, 2, 1]
[1, 2, 1, 1, 2]
[0, 1, 1, 0, 2]
With community centers:
((1, 2), 1)
Final region:
[0, 0, 1, 1, 0]
[0, 1, 1, 2, 0]
[0, 0, 2, 2, 0]
[0, 1, 0, 0, 1]
[0, 0, 0, 0, 1]
Final language state frequencies: (15, 7, 3)
Running simulation_sweep
You can run your simulation_sweep function by hand in IPython.
In [3]: import language, utility
In [4]: region, centers = utility.read_grid("tests/writeup-grid-with-cc.txt")
In [5]: Bs = [0.6, 0.8, 1.0, 1.2, 1.4]
In [6]: language.simulation_sweep(region, 1, 0.6, Bs, 1.6, centers, 5)
Out[6]: [(3, 16, 6), (3, 16, 6), (14, 6, 5), (18, 4, 3), (18, 4, 3)]
Notice that we we fi nd the same results as Beltran, et al. As the value of B gets large, most speakers move into language state 0.
Automated testing
In the “Running your program” section above, we described how to use IPython to test out your functions as you write them, as well as how to run your run_simulation function from the command line.
We have also provided test code for your run_simulation and simulation_sweep functions. We will be using py.test to test your code. You can run our run_simulation test from a Linux terminal window with the command:
$ py.test -vx test_run_simulation.py
When a test fails, the output will include instructions on how to run the test in IPython.
We have also provided test code for simulation_sweep . You can run these tests from the command line with:
$ py.test -vx test_simulation_sweep.py
You can fi nd detailed information about running automated tests with py.test in the Testing Your Code (../../resources/testing.html#testing-your-code) page.



         
加QQ：99515681  WX：codinghelp  Email: 99515681@qq.com
