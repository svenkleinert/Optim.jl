# Automatic Differentiation
As mentioned in the [Minimizing a function](../user/minimization.md) section,
it is possible to avoid passing gradients even when using gradient based methods.
This is because Optim will call the finite central differences
functionality in
[DiffEqDiffTools.jl](https://github.com/JuliaDiffEq/DiffEqDiffTools.jl)
in those cases.
The advantages are clear: you do not have to write the gradients yourself, and
it works for any function you can pass to Optim. However, there is another good
way of making the computer provide gradients: *automatic
differentiation*. Again, the advantage is that you can easily get
gradients from the objective function alone.
As opposed to finite difference, these gradients are exact and we also get Hessians for
Newton's method. They can perform better than a finite differences scheme, depending
on the exact problem.
The disadvantage is that the objective function has
to be written using only Julia code, so no calls to BLAS or Fortran functions.

Let us consider the Rosenbrock example again.
```julia
function f(x)
    return (1.0 - x[1])^2 + 100.0 * (x[2] - x[1]^2)^2
end

function g!(G, x)
    G[1] = -2.0 * (1.0 - x[1]) - 400.0 * (x[2] - x[1]^2) * x[1]
    G[2] = 200.0 * (x[2] - x[1]^2)
end

function h!(H, x)
    H[1, 1] = 2.0 - 400.0 * x[2] + 1200.0 * x[1]^2
    H[1, 2] = -400.0 * x[1]
    H[2, 1] = -400.0 * x[1]
    H[2, 2] = 200.0
end

initial_x = zeros(2)
```
Let us see if BFGS and Newton's Method can solve this problem with the functions
provided.
```jlcon
julia> Optim.minimizer(optimize(f, g!, h!, initial_x, BFGS()))
2-element Array{Float64,1}:
 1.0
 1.0

julia> Optim.minimizer(optimize(f, g!, h!, initial_x, Newton()))

2-element Array{Float64,1}:
 1.0
 1.0
```
This is indeed the case. Now let us use finite differences for BFGS.
```jlcon
julia> Optim.minimizer(optimize(f, initial_x, BFGS()))
2-element Array{Float64,1}:
 1.0
 1.0
```
Still looks good. Returning to automatic differentiation, let us try both solvers using this
method.  We enable [forward mode](https://github.com/JuliaDiff/ForwardDiff.jl) automatic
differentiation by using the `autodiff = :forward` keyword.
```jlcon
julia> Optim.minimizer(optimize(f, initial_x, BFGS(); autodiff = :forward))
2-element Array{Float64,1}:
 1.0
 1.0

julia> Optim.minimizer(optimize(f, initial_x, Newton(); autodiff = :forward))
2-element Array{Float64,1}:
 1.0
 1.0
```
Indeed, the minimizer was found, without providing any gradients or Hessians.
