# CME 257 Homework 4
Due Sunday 11/10 at 11:59 pm.

Please submit the assignment in a IJulia notebook (.ipynb). Name your ipynb "lastname_hw4.ipynb". This assignment shouldn't take you more than 90 minutes.

Email your .ipynb to jmblpati@stanford.edu with subject "cme 257 hw 4 submission"

---

In this homework you'll implement a strange type of matrix multiplication in Julia, and use the techniques from lecture 5 to optimize your implementation. You will also make use of PyCall to call a Python package for doing graph theory, and finally use your strange matrix-vector product to compute distances in graphs.

This assignment shouldn't take you more than 90 minutes.

---

* (Part 1) We define the [*min-plus*](https://en.wikipedia.org/wiki/Min-plus_matrix_multiplication) on matrices as follows. Given two (n by n) matrices A and B, we want an output matrix C where C[i,j] = min<sub>k</sub> (A[i,k] + B[k,j]). In other words, to construct C[i,j] we take the i<sup>th</sup> row from A and the j<sup>th</sup> column from B, and find k to minimize A[i,k] + B[k,j]. Write a naive implementation of min-plus multiplication in Julia.

* (Part 2) Using some of ^e tricks from Lecture 5, optimize your implementation of the min-plus product. Your code should use some of the vectorization macros, and you should experiment with pre-allocating memory. Keep in mind some of the things we discussed about the speed directly modifying the entries of an array versus using a local variable. Benchmark your implementations on random n by n integer matrices with n = 10,50,100,500,1000,2000. 

* (Part 3) Now, we will generate graph adjacency matrices using Python and PyCall. To make PyCall find packages installed in our system-wide python3 installation, we add to Julia's ENV list and rebuild Pycall. From the terminal, run the command
```bash
which python3
```
and take the output-- it should look like `/usr/local/bin/python3` or   `/usr/bin/python3` on Mac or Linux. Run the following code
```julia
using PyCall
ENV["PYTHON"] = {Whatever string that you got fron the previous step}
using Pkg
Pkg.build("PyCall")
```
and then restart your Julia notebook's kernel.

* (Part 4) Using NetworkX, Numpy, and Scipy through PyCall, we will write a function which generates a random graph and returns its adjacency matrix and diameter. Here is some starter code.
```julia
using PyCall
nx = pyimport("networkx")
np = pyimport("numpy")
sp = pyimport("scipy.sparse")
function generate_random_graph(n)
    G = nx.gnm_random_graph(n,5*n)
    return G
end
function adjacency_from_graph(G)
    *your code here*
end
```
So far, the above code returns a random Erdos-Renyi random graph G with n vertices and 5n edges. Extend the function `adjacency_from_graph` to return the adjacency matrix of G represented as a dense matrix.
(**Hint**: You may find the functions `nx.adjacency_matrix()` and `sp.csr_matrix.todense()` to be helpful.)

* (Part 5) Now, we're going to use our special min-plus product routine to compute the diameter of random graphs. Here is some Julia code to compute the diameter of random graphs generated by the code in part 3:
```julia 
g = nx.gnm_random_graph(n,n)
function python_diameter(g)
    if nx.is_connected(g)
        return(nx.diameter(g))
    else
        return(g.number_of_nodes()+1)
    end
end
```
If A is the standard adjacency matrix of an n node graph, let B be the matrix where every 0 entry in A that is not on the diagonal is replaced with n+1. Hence if A was
```julia

 0  1  0  0  1
 1  0  1  0  1
 0  1  0  1  1
 0  0  1  0  0
 1  1  1  0  0
 ```
 , B is 
 ```julia
 0  1  6  6  1
 1  0  1  6  1
 6  1  0  1  1
 6  6  1  0  6
 1  1  1  6  0
```
Write a function to turn a normal adjacency matrix into this "modified" adjacency matrix. It turns out that if G is an n-node graph with adjacency matrix A and modified adjacency matrix B, the diameter of G is precisely the maximum value in the n<sup>th</sup> power of B (or n+1 if G is not connected). In other words, if we take B, copy it to another matrix C, and multiply B by C n times using the min-plus product, we can compute the diameter of G! Implement this using your optimized min-plus product implementation, and compare its output with `python_diameter`.
(**Note**: it is important to use the min plus product here! Using normal matrix multiplication will just cause B to go off to infinity.)

* (Part 6 (Bonus)) It also turns out that the min-plus product is associative. Thus, we can compute the n<sup>th</sup> min-plus power by [repeated squaring](https://en.wikipedia.org/wiki/Exponentiation_by_squaring). Implement this in Julia, and compare its running time to the naive method in Part 5.
