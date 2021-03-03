# Julia-Cheatsheet
A Julia cheatsheet including some tricks and recommended styles


## Array indexing
### Getting index and value
Generic way to access index and element of an array.
```julia
julia> using OffsetArrays

julia> x = randn((2,2));

julia> x_offset = OffsetArray(copy(x), 0:1, 0:1);

julia> for (i, p) in pairs(x)
                 @show i, p
             end
(i, p) = (CartesianIndex(1, 1), -0.5065360216287599)
(i, p) = (CartesianIndex(2, 1), 0.5615767534606946)
(i, p) = (CartesianIndex(1, 2), 0.5510864594685746)
(i, p) = (CartesianIndex(2, 2), -1.2135138442329085)

julia> for (i, p) in pairs(x_offset)
                 @show i, p
             end
(i, p) = (CartesianIndex(0, 0), -0.5065360216287599)
(i, p) = (CartesianIndex(1, 0), 0.5615767534606946)
(i, p) = (CartesianIndex(0, 1), 0.5510864594685746)
(i, p) = (CartesianIndex(1, 1), -1.2135138442329085)
```

### Iterating over a dimension
```julia
julia> for row in axes(x_offset, 1)
           x_offset[row, :] .= 0
       end

julia> x_offset
2×2 OffsetArray(::Matrix{Float64}, 0:1, 0:1) with eltype Float64 with indices 0:1×0:1:
 0.0  0.0
 0.0  0.0
```


## Reversing Array
From Julia 1.6>= you can reverse multiple dimensions
```julia
julia> M=[i+j for i in -3:0, j in -1:1]
4×3 Matrix{Int64}:
 -4  -3  -2
 -3  -2  -1
 -2  -1   0
 -1   0   1

julia> reverse(M; dims=(1,2))
4×3 Matrix{Int64}:
  1   0  -1
  0  -1  -2
 -1  -2  -3
 -2  -3  -4
```

## Taking N dimensional slice of an array
We use `selectdim`. First argument array, second argument dimension, third argument position in that dimension.,
```julia 
julia> x = [i+3*j+9*k for i=0:2,j=0:2,k=0:2 ]
3×3×3 Array{Int64, 3}:
[:, :, 1] =
 0  3  6
 1  4  7
 2  5  8

[:, :, 2] =
  9  12  15
 10  13  16
 11  14  17

[:, :, 3] =
 18  21  24
 19  22  25
 20  23  26

julia> selectdim(x, 3, 1)
3×3 view(::Array{Int64, 3}, :, :, 1) with eltype Int64:
 0  3  6
 1  4  7
 2  5  8

julia> selectdim(x, 1, 3)
3×3 view(::Array{Int64, 3}, 3, :, :) with eltype Int64:
 2  11  20
 5  14  23
 8  17  26
```
