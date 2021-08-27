# Julia-Cheatsheet
A Julia cheatsheet including some tricks and recommended styles

## `cispi` (Julia 1.6.2)
```julia
julia> using BenchmarkTools

julia> x = randn(ComplexF32, (1024, 200));

julia> @btime exp.(1im .* eltype($x)(π) .* $x);
  3.194 ms (2 allocations: 1.56 MiB)
  
julia> @btime cis.(eltype($x)(π) .* $x)
  3.103 ms (2 allocations: 1.56 MiB)
  
julia> @btime cispi.($x);
  4.825 ms (2 allocations: 1.56 MiB)
  
  
julia> x = randn(Float32, (1024, 200));

julia> @btime exp.(1im .* eltype($x)(π) .* $x);
  3.357 ms (2 allocations: 1.56 MiB)

julia> @btime cis.(eltype($x)(π) .* $x)
  2.952 ms (2 allocations: 1.56 MiB)

julia> @btime cispi.($x);
  2.234 ms (2 allocations: 1.56 MiB)
```

## FFT and CUDA
* Julia 1.6.2 and AMD Ryzen 5 5600X and RTX 2060 Super 8GB.
```julia
julia> using FFTW, BenchmarkTools

julia> using CUDA

julia> x = randn(Float32, (50, 50, 50))

julia> x_c = CuArray(x);

julia> p = plan_fft(x, (2,3));

julia> @benchmark p * x
BenchmarkTools.Trial: 7102 samples with 1 evaluation.
 Range (min … max):  590.614 μs …   1.529 ms  ┊ GC (min … max): 0.00% … 0.00%
 Time  (median):     673.503 μs               ┊ GC (median):    0.00%
 Time  (mean ± σ):   701.655 μs ± 112.999 μs  ┊ GC (mean ± σ):  3.02% ± 8.31%

      ▂▁▂██▅▃▃▂▁  ▁                                         ▂▂  ▂
  ▃▄▄████████████▇██▇▆▅▅▃▃▅▄▄▃▃▄▁▃▁▁▁▁▁▃▁▄▁▄▄▄▄▁▃▁▁▁▁▃▁▃▁▁▃████ █
  591 μs        Histogram: log(frequency) by time       1.21 ms <

 Memory estimate: 1.91 MiB, allocs estimate: 4.

julia> p = plan_fft(x, (1,2));

julia> julia> @benchmark p * x
BenchmarkTools.Trial: 9601 samples with 1 evaluation.
 Range (min … max):  464.915 μs …   1.886 ms  ┊ GC (min … max): 0.00% … 57.57%
 Time  (median):     482.785 μs               ┊ GC (median):    0.00%
 Time  (mean ± σ):   518.510 μs ± 161.075 μs  ┊ GC (mean ± σ):  6.30% ± 12.13%

  ██▆▄                                                  ▁▂▁     ▂
  █████▅▁▁▃▁▁▁▁▁▁▁▁▃▁▃▃▄▄▆▅▅▁▃▃▁▁▅▅▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▅████▇█▇ █
  465 μs        Histogram: log(frequency) by time        1.3 ms <

 Memory estimate: 1.91 MiB, allocs estimate: 4.
 
 julia> p_c = plan_fft(x_c, (2,3));

julia> @benchmark (CUDA.@sync p_c * x_c)
BenchmarkTools.Trial: 10000 samples with 1 evaluation.
 Range (min … max):  54.940 μs …   8.038 ms  ┊ GC (min … max): 0.00% … 34.24%
 Time  (median):     56.229 μs               ┊ GC (median):    0.00%
 Time  (mean ± σ):   60.960 μs ± 169.297 μs  ┊ GC (mean ± σ):  2.16% ±  0.78%

  ▁▆██▇▄▂▁                                                     ▂
  █████████▇▆▅▅▅▅▅▅▄▅▄▄▄▄▄▁▄▁▄▁▃▃▄▃▁▁▁▁▃▃▁▃▁▃▄▁▁▁▃▁▁▃▁▁▁▄▃▁▁▃▄ █
  54.9 μs       Histogram: log(frequency) by time      79.6 μs <

 Memory estimate: 2.19 KiB, allocs estimate: 54.
 
julia> p_c = plan_fft(x_c, (1,2));

julia> @benchmark (CUDA.@sync p_c * x_c)
BenchmarkTools.Trial: 10000 samples with 1 evaluation.
 Range (min … max):  36.780 μs …   6.213 ms  ┊ GC (min … max): 0.00% … 20.93%
 Time  (median):     37.960 μs               ┊ GC (median):    0.00%
 Time  (mean ± σ):   41.191 μs ± 128.370 μs  ┊ GC (mean ± σ):  1.61% ±  0.52%

                    ▁▂▂▃▄▇▆▆█▇▇▇▅▇▆▄▄▅▄▃▂▁                      
  ▂▁▂▂▂▂▂▃▃▃▃▃▄▅▆▆▇██████████████████████████▇▆▆▅▅▄▄▄▄▃▃▃▃▃▂▃▃ ▅
  36.8 μs         Histogram: frequency by time         39.2 μs <

 Memory estimate: 2.28 KiB, allocs estimate: 60.
```

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


## Bugs
* GTK.jl + Threads causes [major slowdowns](https://github.com/JuliaGraphics/Gtk.jl/issues/503)


## Nice Packages
### Speed!
* For fast array operations see [Tullio.jl](https://github.com/mcabbott/Tullio.jl)
### Development
* https://github.com/JuliaTesting/TestEnv.jl

### Arrays Helpers
* [https://github.com/SciML/RecursiveArrayTools.jl](https://github.com/SciML/RecursiveArrayTools.jl)
* https://github.com/jonniedie/ComponentArrays.jl
