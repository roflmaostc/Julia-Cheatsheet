# Julia-Cheatsheet
A Julia cheatsheet including some tricks and recommended styles

## Complex Plots
```
# ╔═╡ 487fbad4-dd5d-11ee-369c-2b789fd929d8
using Plots, ImageShow, ImageIO

# ╔═╡ 818eb241-511c-4ed8-b8f3-d5660916fcb3
img = randn(ComplexF32, (50, 30))

# ╔═╡ 8de60652-51d6-41ea-91c4-adc5f5117464
y = range(0, 1, 50)

# ╔═╡ aa6e86d3-9993-4ddb-8d3f-5710d09656b5
x = range(0, 1, 30)

# ╔═╡ 145aec23-f6f4-4043-a171-a3b3225a7af3
heatmap(y, x, simshow(cispi.(randn((30, 50)))), xlim=(0,1), 
	zcolor=range(0,2π,100),  colorbar_title="phase",
	colorbar_ticks=[0,π/2, π, π * 3 / 2, 2π],
	color = palette(:hsv, length(unique(range(0, 1, 100)))))
```

## Irrationals such as `pi`
Pay attention if you work with Irrationals such as `\pi` or `\euler` because of this:
```julia
julia> x = randn(Float32, (2, 2));

julia> x ./ (2π)    # return type is no longer Matrix{Float32}
2×2 Matrix{Float64}:
 0.0696637  -0.0217089
 0.115218   -0.0953945

julia> x ./ π
2×2 Matrix{Float32}:
  0.422652   -0.122149
 -0.0879406   0.474876

julia> x ./ (eltype(x)(2π))   # workaround
2×2 Matrix{Float32}:
 0.0696637  -0.0217089
 0.115218   -0.0953945
 ```
 The Array element type changes. Therefore always convert your Irrational! Often you can do in functions `T(pi)` if you have an 

## Fast closures (very important for performance!)
* [See here](https://docs.julialang.org/en/v1/manual/performance-tips/#man-performance-captured)
Second structure is better
```julia
function gen_f(x)
    if true
        y = x .+ 1
    end 

    function f(a)
        a + y 
    end 
    return f
end

function gen_f_safe(x)
    if true
        y = x .+ 1
    end 

    f = let y = y
        f(a, y=y) = a + y
    end 
    return f
end
```

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

## Convert a `Vector{Vector{Float64}}` to a `Matrix{Float64}`
* `reduce(hcat, x)` works
* don't use `hcat(x...)` since that is inefficient for a long Vector.

## NTuple
* Don't call `Tuple` on generator

```julia
julia> f(x) = Tuple(i for i=1:length(x))
f (generic function with 1 method)

julia> @code_warntype f((1,2,3,4))
MethodInstance for f(::NTuple{4, Int64})
  from f(x) in Main at REPL[5]:1
Arguments
  #self#::Core.Const(f)
  x::NTuple{4, Int64}
Body::Tuple{Vararg{Int64}}
1 ─ %1 = Main.length(x)::Core.Const(4)
│   %2 = (1:%1)::Core.Const(1:4)
│   %3 = Base.Generator(Base.identity, %2)::Core.Const(Base.Generator{UnitRange{Int64}, typeof(identity)}(identity, 1:4))
│   %4 = Main.Tuple(%3)::Tuple{Vararg{Int64}}
└──      return %4


julia> f(x) = ntuple(i -> i, 1:length(x))
f (generic function with 1 method)

julia> @code_warntype f((1,2,3,4))
MethodInstance for f(::NTuple{4, Int64})
  from f(x) in Main at REPL[7]:1
Arguments
  #self#::Core.Const(f)
  x::NTuple{4, Int64}
Locals
  #1::var"#1#2"
Body::Union{}
1 ─      (#1 = %new(Main.:(var"#1#2")))
│   %2 = #1::Core.Const(var"#1#2"())
│   %3 = Main.length(x)::Core.Const(4)
│   %4 = (1:%3)::Core.Const(1:4)
│        Main.ntuple(%2, %4)
└──      Core.Const(:(return %5)



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

### Benchmarking 
```
using FFTW, CUDA, BenchmarkTools

FFTW.set_num_threads(1)
function f() 
        for i in [10, 30, 50, 100, 256, 512, 1024, 1500, 2048, 2500, 3000, 4096, 5000, 6200]
                x = randn(ComplexF32, (i, i)) 
                x_copy = similar(x) 
                xc = CuArray(x)
                xc_copy = similar(xc)
                p = plan_fft!(similar(x), flags=FFTW.MEASURE)
                pc = plan_fft!(similar(xc))
                print("Size:\t\t($i, $i)\n")
                print("CPU FFT:\t")
                @btime $p * $x
                print("CPU exp.(i.*x):\t")
                @btime $x_copy .= exp.(1im .* $x) 
                print("GPU FFT:\t")
                @btime CUDA.@sync $pc * $xc 
                print("GPU exp.(i.*x):\t")
                @btime $xc_copy .= exp.(1im .* $xc)
                print("\n")
        end 
end

julia> f(); g()
Size:           (10, 10)
CPU FFT:          100.634 ns (0 allocations: 0 bytes)
CPU exp.(i.*x):   84.258 ns (0 allocations: 0 bytes)
GPU FFT:          6.718 μs (0 allocations: 0 bytes)
GPU exp.(i.*x):   5.907 μs (24 allocations: 1.61 KiB)

Size:           (30, 30)
CPU FFT:          2.629 μs (0 allocations: 0 bytes)
CPU exp.(i.*x):   802.402 ns (0 allocations: 0 bytes)
GPU FFT:          7.814 μs (0 allocations: 0 bytes)
GPU exp.(i.*x):   6.794 μs (25 allocations: 1.62 KiB)

Size:           (50, 50)
CPU FFT:          6.551 μs (0 allocations: 0 bytes)
CPU exp.(i.*x):   2.010 μs (0 allocations: 0 bytes)
GPU FFT:          8.273 μs (0 allocations: 0 bytes)
GPU exp.(i.*x):   6.755 μs (25 allocations: 1.62 KiB)

Size:           (100, 100)
CPU FFT:          21.653 μs (0 allocations: 0 bytes)
CPU exp.(i.*x):   7.412 μs (0 allocations: 0 bytes)
GPU FFT:          8.459 μs (0 allocations: 0 bytes)
GPU exp.(i.*x):   6.775 μs (25 allocations: 1.62 KiB)

Size:           (256, 256)
CPU FFT:          131.983 μs (0 allocations: 0 bytes)
CPU exp.(i.*x):   47.651 μs (0 allocations: 0 bytes)
GPU FFT:          9.982 μs (0 allocations: 0 bytes)
GPU exp.(i.*x):   7.779 μs (25 allocations: 1.62 KiB)

Size:           (512, 512)
CPU FFT:          694.378 μs (0 allocations: 0 bytes)
CPU exp.(i.*x):   188.841 μs (0 allocations: 0 bytes)
GPU FFT:          19.201 μs (0 allocations: 0 bytes)
GPU exp.(i.*x):   20.571 μs (25 allocations: 1.62 KiB)

Size:           (1024, 1024)
CPU FFT:          2.890 ms (0 allocations: 0 bytes)
CPU exp.(i.*x):   798.956 μs (0 allocations: 0 bytes)
GPU FFT:          116.860 μs (0 allocations: 0 bytes)
GPU exp.(i.*x):   62.641 μs (25 allocations: 1.62 KiB)

Size:           (1500, 1500)
CPU FFT:          8.380 ms (0 allocations: 0 bytes)
CPU exp.(i.*x):   1.806 ms (0 allocations: 0 bytes)
GPU FFT:          404.556 μs (52 allocations: 3.08 KiB)
GPU exp.(i.*x):   133.701 μs (25 allocations: 1.62 KiB)

Size:           (2048, 2048)
CPU FFT:          17.031 ms (0 allocations: 0 bytes)
CPU exp.(i.*x):   3.399 ms (0 allocations: 0 bytes)
GPU FFT:          540.220 μs (52 allocations: 3.08 KiB)
GPU exp.(i.*x):   289.344 μs (77 allocations: 4.70 KiB)

Size:           (2500, 2500)
CPU FFT:          35.288 ms (0 allocations: 0 bytes)
CPU exp.(i.*x):   5.125 ms (0 allocations: 0 bytes)
GPU FFT:          891.219 μs (52 allocations: 3.08 KiB)
GPU exp.(i.*x):   398.880 μs (77 allocations: 4.70 KiB)

Size:           (30, 30, 30)
CPU FFT:          120.564 μs (0 allocations: 0 bytes)
CPU exp.(i.*x):   24.045 μs (0 allocations: 0 bytes)
GPU FFT:          11.272 μs (0 allocations: 0 bytes)
GPU exp.(i.*x):   7.706 μs (25 allocations: 1.78 KiB)

Size:           (50, 50, 50)
CPU FFT:          533.770 μs (0 allocations: 0 bytes)
CPU exp.(i.*x):   101.217 μs (0 allocations: 0 bytes)
GPU FFT:          17.654 μs (0 allocations: 0 bytes)
GPU exp.(i.*x):   10.658 μs (25 allocations: 1.78 KiB)

Size:           (100, 100, 100)
CPU FFT:          3.563 ms (0 allocations: 0 bytes)
CPU exp.(i.*x):   794.678 μs (0 allocations: 0 bytes)
GPU FFT:          203.294 μs (51 allocations: 3.00 KiB)
GPU exp.(i.*x):   60.779 μs (25 allocations: 1.78 KiB)

Size:           (256, 256, 256)
CPU FFT:          72.164 ms (0 allocations: 0 bytes)
CPU exp.(i.*x):   14.064 ms (0 allocations: 0 bytes)
GPU FFT:          2.692 ms (52 allocations: 3.08 KiB)
GPU exp.(i.*x):   990.830 μs (77 allocations: 4.86 KiB)

Size:           (331, 331, 331)
CPU FFT:          1.324 s (0 allocations: 0 bytes)
CPU exp.(i.*x):   30.814 ms (0 allocations: 0 bytes)
GPU FFT:          15.145 ms (52 allocations: 3.08 KiB)
GPU exp.(i.*x):   2.241 ms (78 allocations: 4.80 KiB)


Size:           (512, 512, 512)
CPU FFT:          855.874 ms (0 allocations: 0 bytes)
CPU exp.(i.*x):   111.599 ms (0 allocations: 0 bytes)
GPU FFT:          20.587 ms (52 allocations: 3.08 KiB)
GPU exp.(i.*x):   7.676 ms (80 allocations: 4.91 KiB)


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


### Broadcast Shape




```julia
function f(sa::NTuple{N, T}, sb::NTuple{N, T}) where {N, T}
    function f(i)
        if sb[i] == sa[i]
            return sb[i]    
        elseif sb[i] == 1 || sa[i] == 1
            return max(sa[i], sb[i])
        else
            error("Broadcast error")
        end
    end
    return ntuple(i -> f(i), N)
end


function f(sa::NTuple{N, T}, sb::NTuple{M, T}) where {N, M, T}
    if M < N
        return f(sb, sa)
    else
        return f(ntuple(i -> i ≤ N ? sa[i] : 1, M) ,sb)
    end
end 
