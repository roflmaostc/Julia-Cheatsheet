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
```
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

