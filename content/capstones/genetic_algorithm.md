---
title: Genetic algorithm
weight: 1
status: construction
packages:
  - Plots
  - StatsBase
  - Statistics
concepts:
  - arrays
  - control flow
---

Intro to GA

Definition of the problem: MacBeth line

````julia
problem = lowercase("What, you egg? Young fry of treachery!")
````





Initial solution

````julia
using StatsBase
search_space = split("abcdefghijklmnopqrstuvwxyz !?,.-", "")
initial_guess = reduce(*, sample(search_space, length(problem), replace=true))
````





fitness function

````julia
using Statistics

function ω(guess, solution)
  return mean(split(guess, "") .== split(solution, ""))
end

ω(initial_guess, problem)
````


````
0.0
````





Generating a lot of initial guesses

````julia
initial_guesses = [reduce(*, sample(search_space, length(problem), replace=true)) for i in 1:500];
````





We can view the fitness distribution:

````julia
using Plots
scatter(sort(ω.(initial_guesses, problem)), leg=false, c=:grey, msw=0.0)
````


{{< figure src="../figures/genetic_algorithm_5_1.svg" title="TODO"  >}}


Picking the next generation -- pair of parents, recombination at random point,
then mutation:

````julia
Ω = ω.(initial_guesses, problem)
parents = sample(initial_guesses, StatsBase.weights(Ω), 2; replace=false)
cutoff = rand(1:length(problem))
offspring = first(parents)[1:cutoff] * last(parents)[cutoff+1:end]
@info ω(offspring, problem)
````

