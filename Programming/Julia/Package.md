---
owner: Daniele Andreis
created: November 08, 2025 – 8:43 AM
updated: 2025-11-08
tags: [Julia, package]
---


# Reloading a Julia Package After Changes

When developing a Julia package and testing it in the REPL (Read-Eval-Print Loop), it is common to make changes to your package code and then reload it to see the effects of those changes. Here’s a step-by-step process on how to effectively reload your Julia package in the REPL after making changes:

### Step-by-Step Guide

1. **Start the Julia REPL**:
Open your terminal and start the Julia REPL by typing `julia`.
2. **Activate Your Package Environment**:
Navigate to the directory containing your package and activate the environment.
    
    ```julia
    cd("/path/to/your/package")
    using Pkg
    Pkg.activate(".")
    ```
    
3. **Develop the Package**:
    
    Inform Julia that you are actively developing this package. This makes it easier to reload changes.
    
    ```julia
      Pkg.develop(path=".")  # Only needed once per session or project setup 
    ```
    
4. **Using `Revise.jl`**:
The `Revise.jl` package is very useful for automatically reloading changes made to your package without needing to restart the REPL. Install and use it as follows:
    
    ```
    using Pkg
    Pkg.add("Revise")  # Install Revise.jl if not already installed
    using Revise
    ```
    
5. **Using `Revise.jl`**:
The `Revise.jl` package is very useful for automatically reloading changes made to your package without needing to restart the REPL. Install and use it as follows:
    
    ```
    using YourPackageName
    ```
    
6. **Make Changes and Reload Automatically**
With Revise.jl  loaded, any changes you make to the package source code will be automatically tracked and updated in the REPL. There is no need to manually reload the package.

Here’s an example workflow:

```julia
# Start Julia REPL
julia

# Activate the package environment
julia> cd("/path/to/your/package")
julia> using Pkg
julia> Pkg.activate(".")

# Inform Julia that you are developing this package
julia> Pkg.develop(path=".")

# Load Revise.jl
julia> using Revise

# Load your package
julia> using YourPackageName

# Now, any changes made to the source code of YourPackageName will be automatically reloaded
# by Revise.jl. You can modify your code and immediately see the changes reflected in the REPL.

```