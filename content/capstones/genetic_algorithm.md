---
title: Genetic algorithm
weight: 1
status: draft
packages:
  - StatsPlots
  - StatsKit
  - Statistics
concepts:
  - data frames
  - generic code
  - type system
---

Genetic algorithm is a heuristic that takes heavy inspiration from evolutionary
biology, to explore a space of parameters rapidly and converge to an optimum.
Every solution is a "genome", and the combinations can undergo mutation and
recombination. By simulating a process of reproduction, over sufficiently many
generation, this heuristic usually gives very good results. It is also simple to
implement, and this is what we will do!

A genetic algorithm works by measuring the fitness of a solution (*i.e.* its fit
to a problem we have defined). We can then pick solution for "reproduction",
which involves recombination at various points in the "genome", and finally a
step for the mutation of offspring. There are an almost infinite number of
variations at each of these steps, but we will limit ourselves to a simple case
here.

Specifically, we will use the `DataFrames` and `CSV` package (installed in
`StatsKit`) to read a table containing metabolic rates of species, and then use
genetic algorithm to try and find out the scaling exponent linking mass to the
field metabolic rate.

[Source of data](https://besjournals.onlinelibrary.wiley.com/doi/full/10.1111/1365-2656.12086)

````julia
using StatsKit
using StatsPlots
url = "http://sciencecomputing.io/data/metabolicrates.csv"
tmp = download(url)
rates = CSV.read(tmp)
rates[1:5,:]
````



<table class="data-frame"><thead><tr><th></th><th>Class</th><th>Order</th><th>Family</th><th>Genus</th><th>Species</th><th>Study</th><th>M (kg)</th><th>FMR (kJ / day)</th></tr><tr><th></th><th>String</th><th>String</th><th>String</th><th>String</th><th>String</th><th>String</th><th>Float64</th><th>Float64</th></tr></thead><tbody><p>5 rows × 8 columns</p><tr><th>1</th><td>Mammalia</td><td>Carnivora</td><td>Odobenidae</td><td>Odobenus</td><td>rosmarus</td><td>Acquarone et al 2006</td><td>1370.0</td><td>345000.0</td></tr><tr><th>2</th><td>Mammalia</td><td>Carnivora</td><td>Odobenidae</td><td>Odobenus</td><td>rosmarus</td><td>Acquarone et al 2006</td><td>1250.0</td><td>417400.0</td></tr><tr><th>3</th><td>Aves</td><td>Procellariiformes</td><td>Diomedeidae</td><td>Diomedea</td><td>exulans</td><td>Adams et al 1986</td><td>7.4</td><td>3100.0</td></tr><tr><th>4</th><td>Aves</td><td>Procellariiformes</td><td>Diomedeidae</td><td>Diomedea</td><td>exulans</td><td>Adams et al 1986</td><td>6.95</td><td>2898.0</td></tr><tr><th>5</th><td>Aves</td><td>Procellariiformes</td><td>Diomedeidae</td><td>Diomedea</td><td>exulans</td><td>Adams et al 1986</td><td>8.9</td><td>3528.0</td></tr></tbody></table>



Let's have a look at the names of the columns:

````julia
names(rates)
````


````
8-element Array{Symbol,1}:
 :Class                  
 :Order                  
 :Family                 
 :Genus                  
 :Species                
 :Study                  
 Symbol("M (kg)")        
 Symbol("FMR (kJ / day)")
````





We will replace the last two names:

````julia
rename!(rates, names(rates)[end-1] => :mass)
rename!(rates, names(rates)[end] => :fmr)
rates = rates[rates[:mass].<1e3,:]
rates[1:10,:]
````



<table class="data-frame"><thead><tr><th></th><th>Class</th><th>Order</th><th>Family</th><th>Genus</th><th>Species</th><th>Study</th><th>mass</th><th>fmr</th></tr><tr><th></th><th>String</th><th>String</th><th>String</th><th>String</th><th>String</th><th>String</th><th>Float64</th><th>Float64</th></tr></thead><tbody><p>10 rows × 8 columns</p><tr><th>1</th><td>Aves</td><td>Procellariiformes</td><td>Diomedeidae</td><td>Diomedea</td><td>exulans</td><td>Adams et al 1986</td><td>7.4</td><td>3100.0</td></tr><tr><th>2</th><td>Aves</td><td>Procellariiformes</td><td>Diomedeidae</td><td>Diomedea</td><td>exulans</td><td>Adams et al 1986</td><td>6.95</td><td>2898.0</td></tr><tr><th>3</th><td>Aves</td><td>Procellariiformes</td><td>Diomedeidae</td><td>Diomedea</td><td>exulans</td><td>Adams et al 1986</td><td>8.9</td><td>3528.0</td></tr><tr><th>4</th><td>Aves</td><td>Procellariiformes</td><td>Diomedeidae</td><td>Diomedea</td><td>exulans</td><td>Adams et al 1986</td><td>9.6</td><td>3881.0</td></tr><tr><th>5</th><td>Aves</td><td>Procellariiformes</td><td>Diomedeidae</td><td>Diomedea</td><td>exulans</td><td>Adams et al 1986</td><td>9.4</td><td>3830.0</td></tr><tr><th>6</th><td>Aves</td><td>Procellariiformes</td><td>Diomedeidae</td><td>Diomedea</td><td>exulans</td><td>Adams et al 1986</td><td>9.3</td><td>4284.0</td></tr><tr><th>7</th><td>Aves</td><td>Procellariiformes</td><td>Diomedeidae</td><td>Diomedea</td><td>exulans</td><td>Adams et al 1986</td><td>9.35</td><td>3906.0</td></tr><tr><th>8</th><td>Aves</td><td>Procellariiformes</td><td>Diomedeidae</td><td>Diomedea</td><td>exulans</td><td>Adams et al 1986</td><td>8.15</td><td>2386.0</td></tr><tr><th>9</th><td>Aves</td><td>Procellariiformes</td><td>Diomedeidae</td><td>Diomedea</td><td>exulans</td><td>Adams et al 1986</td><td>6.7</td><td>2374.0</td></tr><tr><th>10</th><td>Mammalia</td><td>Carnivora</td><td>Otariidae</td><td>Arctocephalus</td><td>gazella</td><td>Arnould et al 1996</td><td>32.75</td><td>17430.3</td></tr></tbody></table>



Much better! Now let's try to figure out the relationship between the last two
variables:

````julia
scatter(rates[:mass], rates[:fmr], c=:teal, leg=false, msc=:transparent)
xaxis!(:log10, "Mass (kg)")
yaxis!(:log10, "Field Metabolic Rate (kj per day)")
````


{{< figure src="../figures/genetic_algorithm_4_1.png" title="Relationship between field metabolic rate and mass - this is a neat log-log relationship, and so linear regression will give us the exponent."  >}}


Neat! This is a log-log relationship, so we can represent this problem as:

$$\text{log}{10}(\text{FMR}) \propto m\times\text{log}_{10}(\text{M})+b$$

To simplify the problem a little bit, we will average the values within species;
because the relation is log-log, we will average the log of the value (as
opposed to taking the log of the averages).

````julia
rates = by(rates, [:Genus, :Species], [:mass, :fmr] => x -> (mass=mean(log10.(x.mass)), fmr=mean(log10.(x.fmr))))
rates[1:10,:]
````



<table class="data-frame"><thead><tr><th></th><th>Genus</th><th>Species</th><th>mass</th><th>fmr</th></tr><tr><th></th><th>String</th><th>String</th><th>Float64</th><th>Float64</th></tr></thead><tbody><p>10 rows × 4 columns</p><tr><th>1</th><td>Diomedea</td><td>exulans</td><td>0.952967</td><td>3.35304</td></tr><tr><th>2</th><td>Arctocephalus</td><td>gazella</td><td>1.53131</td><td>4.32877</td></tr><tr><th>3</th><td>Sula</td><td>sula</td><td>0.0263231</td><td>3.07421</td></tr><tr><th>4</th><td>Macrotus</td><td>californicus</td><td>-1.88828</td><td>1.33622</td></tr><tr><th>5</th><td>Microtus</td><td>pennsylvanicus</td><td>-1.44936</td><td>1.9947</td></tr><tr><th>6</th><td>Tarsipes</td><td>rostratus</td><td>-2.04591</td><td>1.43081</td></tr><tr><th>7</th><td>Muscicapa</td><td>striata</td><td>-1.84164</td><td>1.71799</td></tr><tr><th>8</th><td>Parus</td><td>major</td><td>-1.74771</td><td>1.98197</td></tr><tr><th>9</th><td>Turdus</td><td>merula</td><td>-1.01773</td><td>2.256</td></tr><tr><th>10</th><td>Hirundo</td><td>tahitica</td><td>-1.85214</td><td>1.86032</td></tr></tbody></table>



We can look at the simplified data:

````julia
scatter(10.0.^rates[:mass], 10.0.^rates[:fmr], c=:teal, leg=false, msc=:transparent)
xaxis!(:log10, "Mass (kg)")
yaxis!(:log10, "Field Metabolic Rate (kj per day)")
````


{{< figure src="../figures/genetic_algorithm_6_1.png" title="Same data, but a little bit less points. In addition, aggregating by species doesn't increase the impact that well sampled species will have on the regression output."  >}}


{{< callout information >}}
In this capstone, we will try to write a very general code for genetic
algorithms. It is possible to come up with a more specific code, but spending
time to think about making code general is almost always wise, since it means we
can seamlessly re-use already written code.
{{< /callout >}}

Genetic algorithms represent the state of the problem as "genomes", which here
will be composed of two genes: $m$ and $b$. There are a few decisions we need to
take already: how large is our initial population, and how much standing
variation do we have?

Just to be a little fancier than usual, we will define a *type* for our genomes:

````julia
mutable struct Genome
  m::Float64
  b::Float64
end
````





{{< callout information >}}
Defining a new type is *absolutely not* necessary. We are only doing it to show
some interesting features of Julia.
{{< /callout >}}

This means that our population will be of the type `Vector{Genome}`. Now, we
will *add methods* to some of Julia's existing function, so we can write code
that will read exactly like native code would:

````julia
import Base: zero
zero(::Type{Genome}) = Genome(0.0, 0.0)
````


````
zero (generic function with 35 methods)
````





Because we have created a `zero` method for the `Genome` type, we can create our
population:

````julia
population_size = 1_000
population = zeros(Genome, population_size)
population[1:5]
````


````
5-element Array{Main.WeaveSandBox0.Genome,1}:
 Main.WeaveSandBox0.Genome(0.0, 0.0)
 Main.WeaveSandBox0.Genome(0.0, 0.0)
 Main.WeaveSandBox0.Genome(0.0, 0.0)
 Main.WeaveSandBox0.Genome(0.0, 0.0)
 Main.WeaveSandBox0.Genome(0.0, 0.0)
````





To have a slightly more pleasing display, we can also overload the `show`
function:

````julia
import Base: show
show(io::IO, g::Genome) = print(io, "ŷ = $(round(g.m; digits=3))×x + $(round(g.b; digits=3))")
population[1:5]
````


````
5-element Array{Main.WeaveSandBox0.Genome,1}:
 ŷ = 0.0×x + 0.0
 ŷ = 0.0×x + 0.0
 ŷ = 0.0×x + 0.0
 ŷ = 0.0×x + 0.0
 ŷ = 0.0×x + 0.0
````





And we are now ready to start. At this point, it is useful to outline the
general structure of the genetic algorithm:

````julia
function GA!(p::Vector{T}, fitness::Function, mutation!::Function; generations=1_000) where {T}
  for generation in 1:generations
    fitnesses = fitness.(p)
    p = sample(p, weights(fitnesses), length(p), replace=true)
    mutation!.(p)
  end
  return nothing
end
````


````
GA! (generic function with 1 method)
````





Wouldn't it be cool if the real code was actually that simple?

What if I told you this *is* the real code? To make it work, we need to do two
things. First, we need to define a `fitness` function, which, given an input
(here, a `Genome`), will return its "score". Second, we need to define a
`mutation!` function, which will *modify* a genome.

The fitness function can be defined as follows (note that we have already taken
the log10 of the mass and FMR earlier):

````julia
const log10fmr = rates[:fmr]
const log10M = rates[:mass]

ŷ(g::Genome, x::Vector{Float64})=  g.m .* x .+ g.b

function fmr_fitness(g::Genome)
  errors = log10fmr .- ŷ(g, log10M)
  sum_of_errors_sq = sum(errors.^2.0)/length(log10fmr)
  return 1.0/sum_of_errors_sq
end
````


````
fmr_fitness (generic function with 1 method)
````





The first thing we do is "extract" the $\text{log}_{10}$ of the FMR and mass,
and turn them into vectors. Then we define a function which uses these vectors,
and does the linear prediction. You may notice that this function uses elements
from "the outside", which have not been passed as arguments, and this may not be
ideal - this is why the `log10...` variables have been declared as `const`: they
will raise a warning if they are changed.

We can check the score of an "empty" genome ($\hat y = 0$):

````julia
fmr_fitness(zero(Genome))
````


````
0.1406618826939854
````





Now, let's define a function for mutations. Because we might want to change the
rate at which parameters evolve, it would be useful to be able to generate
multiple such functions. And so, our first task is to write a function that will
return another function, which will itself modify the `Genome` object. Note that
we `return nothing` at the end, because all of this function does is change an
existing object.

````julia
function normal_error(σm::Float64, σb::Float64)
  function f(g)
    g.m = rand(Normal(g.m, σm))
    g.b = rand(Normal(g.b, σb))
    return nothing
  end
  return f
end
````


````
normal_error (generic function with 1 method)
````





We can test that this all works as expected:

````julia
change_both! = normal_error(0.01, 0.01)
initial_genome = Genome(0.2, 0.5)
change_both!(initial_genome)
initial_genome
````


````
ŷ = 0.211×x + 0.503
````





Let's now define a function to work on the actual problem:

````julia
very_gradual_change! = normal_error(1e-3, 1e-3)
````


````
(::getfield(Main.WeaveSandBox0, Symbol("#f#4")){Float64,Float64}) (generic 
function with 1 method)
````





We have all of the pieces to apply our genetic algorithm. Before starting, it is
always a good idea to try and eyeball the parameters. Judging from the plot we
made early in the lesson, the intercept is probably just around 3, so we can
probably draw it from $\mathcal{N}(\mu=3, \sigma=1)$; the slope seems to be
close to one but a little bit lower, so we can get it from
$\mathcal{N}(\mu=0.75, \sigma=0.2)$:

````julia
population = [Genome(rand(Normal(0.75,0.2)), rand(Normal(3,1))) for i in 1:500]
````


````
500-element Array{Main.WeaveSandBox0.Genome,1}:
 ŷ = 0.573×x + 2.4  
 ŷ = 0.575×x + 3.115
 ŷ = 0.78×x + 4.82  
 ŷ = 0.614×x + 3.223
 ŷ = 0.805×x + 3.464
 ŷ = 1.115×x + 3.242
 ŷ = 0.652×x + 2.365
 ŷ = 0.801×x + 3.243
 ŷ = 0.658×x + 2.869
 ŷ = 0.85×x + 3.477 
 ⋮                   
 ŷ = 0.619×x + 3.89 
 ŷ = 0.587×x + 2.112
 ŷ = 0.693×x + 3.364
 ŷ = 0.978×x + 3.913
 ŷ = 0.714×x + 2.412
 ŷ = 0.453×x + 4.089
 ŷ = 0.674×x + 3.183
 ŷ = 0.831×x + 4.708
 ŷ = 0.434×x + 3.448
````



````julia
GA!(population, fmr_fitness, very_gradual_change!; generations=10000)
````





**_TODO_**

````julia
scatter(10.0.^rates[:mass], 10.0.^rates[:fmr], c=:teal, leg=false, msc=:transparent)
pred = collect(LinRange(10.0.^minimum(rates[:mass]), 10.0.^maximum(rates[:mass]), 100))
mostfit = fmr_fitness.(population) |> findmax |> last
plot!(pred, 10.0.^ŷ(population[mostfit], log10.(pred)), c=:orange, lw=2, ls=:dash)
xaxis!(:log10, "Mass (kg)")
yaxis!(:log10, "Field Metabolic Rate (kj per day)")
````


{{< figure src="../figures/genetic_algorithm_19_1.png" title="Relationship between field metabolic rate and mass - this is a neat log-log relationship, and so linear regression will give us the exponent."  >}}


We can also look at the equation for the most fit genome: ŷ = 0.554×x + 2.83.
