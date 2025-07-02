Title: Voting in Netflix's The Circle
Date: 2024-05-06 00:00:00
Category: Exploration


In January 2020 Netflix released the first season of "The Circle", a reality competition show in which players interact via the show's eponymous social media platform and rate each other on the quality of those interactions. 
I was curious how the ratings system might work so I wrote a Python script to emulate it. This could just as easily have been done in a spreadsheet.

When a ratings session begins each player ranks the other players who are eligible to be rated from last place to first place. 

Not all players can be rated, but usually everyone can rate. This often happens when new players enter The Circle and it creates an opportunity for existing players to form alliances with noobies.

Let's say there are six existing players, we'll call them **Abe**, **Bruce**, **Carla**, **Dylan**, **Emily**, and **Francine**. There are also two new players, **Georgia** and **Henrietta**. They can rate but can't be rated.

In the script, `raters` defines the names of the people who can rate - usually everyone. 
`ratees` defines everyone who can be rated. This is a subset of raters. In the script I just defined it as an array of indices of raters to keep things DRY. 

The ratings matrix is initialized as a `n_ratees` x `n_raters` matrix of zeros of type `int`. 

The script then gathers input from each rater. Who did **Abe** put in 4th place? Who did **Carla** put in 1st place? 

Once the ratings matrix is assembled, getting the final ratings is as simple as getting the sorted per-row sum of the ratings matrix. The lower you're rated the higher your sum will be and vice versa. 

|              | Abe | Bruce | Carla | Dylan | Emily | Francine | Georgia | Henrietta | **SUM** |
| ------------ | --- | ----- | ----- | ----- | ----- | -------- | ------- | --------- | ------- |
| **Abe**      | 0   | 0     | 3     | 1     | 1     | 2        | 0       | 0         | **7**   |
| **Bruce**    | 4   | 0     | 2     | 2     | 2     | 3        | 3       | 1         | **17**  |
| **Carla**    | 3   | 2     | 0     | 0     | 4     | 1        | 2       | 2         | **14**  |
| **Dylan**    | 2   | 1     | 0     | 0     | 3     | 0        | 1       | 3         | **10**  |
| **Emily**    | 0   | 3     | 1     | 0     | 0     | 4        | 5       | 4         | **17**  |
| **Francine** | 1   | 4     | 4     | 4     | 0     | 0        | 4       | 5         | **22**  |

From this we can see that **Francine** came in 6th place, **Emily** and **Bruce** tied for 4th, **Carla** is 3rd, **Dylan** in 2nd and **Abe** came in 1st. 

```Python

import numpy as np

raters = np.array(["Abe","Bruce","Carla","Dylan","Emily","Francine","Georgia","Henrietta"]) #Names of everyone doing ratings
ratees = np.array([0,1,2,3,4,5]) #Indices of names of people being rated

n_raters = len(raters) #Number of raters
n_ratees = len(ratees) #Number of rateees


ratings_matrix = np.zeros((n_ratees,n_raters),dtype = int)


for rater in range(n_raters):
    selection = []
    
    for position in range(0,n_ratees):
        options = []
        print(f"{raters[rater]},\nSelect someone for position {position+1}.\n")
        for r in ratees:
            if r in selection:
                continue
            elif r == rater:
                continue
            else:
                print(f"{r+1} - {raters[ratees[r]]}") 
                options.append(raters[ratees[r]])
        if len(options)==0:
            continue
        ratee = int(input("\nType a number: "))-1
        
        print(f"\nYou put {raters[ratee]} in position {position+1}")
        selection.append(ratee)
        
        ratings_matrix[ratee,rater] = position
        
        
ratings_sum = np.sum(ratings_matrix,axis=1)
ratings = raters[ratees[np.argsort(ratings_sum)]]

for q in range(len(ratings)):
    print(f"Position {q+1}: {ratings[q]}")
```