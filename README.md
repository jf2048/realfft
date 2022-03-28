# RealFFT: Real-to-complex FFT and complex-to-real iFFT based on RustFFT

This library is a wrapper for RustFFT that enables performing FFT of real-valued data.
The API is designed to be as similar as possible to RustFFT.

Using this library instead of RustFFT directly avoids the need of converting real-valued data to complex before performing a FFT.
If the length is even, it also enables faster computations by using a complex FFT of half the length.
It then packs a 2N long real vector into an N long complex vector, which is transformed using a standard FFT.
The FFT result is then post-processed to give only the first half of the complex spectrum, as an N+1 long complex vector.

The iFFT goes through the same steps backwards, to transform an N+1 long complex spectrum to a 2N long real result.

The speed increase compared to just converting the input to a 2N long complex vector
and using a 2N long FFT depends on the length of the input data.
The largest improvements are for long FFTs and for lengths over around 1000 elements there is an improvement of about a factor 2.
The difference shrinks for shorter lengths, and around 30 elements there is no longer any difference.

### Why use real-to-complex FFT?
#### Using a complex-to-complex FFT
A simple way to get the FFT of a real valued vector is to convert it to complex, and use a complex-to-complex FFT.

Let's assume `x` is a 6 element long real vector:
```
x = [x0r, x1r, x2r, x3r, x4r, x5r]
```

We now convert `x` to complex by adding an imaginary part with value zero. Using the notation `(xNr, xNi)` for the complex value `xN`, this becomes:
```
x_c = [(x0r, 0), (x1r, 0), (x2r, 0), (x3r, 0), (x4r, 0), (x5r, 0)]
```

Performing a normal complex FFT, the result of `FFT(x_c)` is:
```
FFT(x_c) = [(X0r, X0i), (X1r, X1i), (X2r, X2i), (X3r, X3i), (X4r, X4i), (X5r, X5i)]
```

But because our `x_c` is real-valued (all imaginary parts are zero), some of this becomes redundant:
```
FFT(x_c) = [(X0r, 0), (X1r, X1i), (X2r, X2i), (X3r, 0), (X2r, -X2i), (X1r, -X1i)]
```

The last two values are the complex conjugates of `X1` and `X2`. Additionally, `X0i` and `X3i` are zero.
As we can see, the output contains 6 independent values, and the rest is redundant.
But it still takes time for the FFT to calculate the redundant values.
Converting the input data to complex also takes a little bit of time.

If the length of `x` instead had been 7, the result would have been:
```
FFT(x_c) = [(X0r, 0), (X1r, X1i), (X2r, X2i), (X3r, X3i), (X3r, -X3i), (X2r, -X2i), (X1r, -X1i)]
```

The result is similar, but this time there is no zero at `X3i`. Also in this case we got the same number of independent values as we started with.

#### Real-to-complex
Using a real-to-complex FFT removes the need for converting the input data to complex.
It also avoids calculating the redundant output values.

The result for 6 elements is:
```
RealFFT(x) = [(X0r, 0), (X1r, X1i), (X2r, X2i), (X3r, 0)]
```

The result for 7 elements is:
```
RealFFT(x) = [(X0r, 0), (X1r, X1i), (X2r, X2i), (X3r, X3i)]
```

This is the data layout output by the real-to-complex FFT, and the one expected as input to the complex-to-real iFFT.

### Scaling
RealFFT matches the behaviour of RustFFT and does not normalize the output of either FFT of iFFT. To get normalized results, each element must be scaled by `1/sqrt(length)`. If the processing involves both an FFT and an iFFT step, it is advisable to merge the two normalization steps to a single, by scaling by `1/length`.

### Documentation

The full documentation can be generated by rustdoc. To generate and view it run:
```
cargo doc --open
```

### Benchmarks

To run a set of benchmarks comparing real-to-complex FFT with standard complex-to-complex, type:
```
cargo bench
```
The results are printed while running, and are compiled into an html report containing much more details.
To view, open `target/criterion/report/index.html` in a browser.

### Example
Transform a vector, and then inverse transform the result.
```rust
use realfft::RealFftPlanner;
use rustfft::num_complex::Complex;
use rustfft::num_traits::Zero;

let length = 256;

// make a planner
let mut real_planner = RealFftPlanner::<f64>::new();

// create a FFT
let r2c = real_planner.plan_fft_forward(length);
// make input and output vectors
let mut indata = r2c.make_input_vec();
let mut spectrum = r2c.make_output_vec();

// Are they the length we expect?
assert_eq!(indata.len(), length);
assert_eq!(spectrum.len(), length/2+1);

// Forward transform the input data
r2c.process(&mut indata, &mut spectrum).unwrap();

// create an iFFT and an output vector
let c2r = real_planner.plan_fft_inverse(length);
let mut outdata = c2r.make_output_vec();
assert_eq!(outdata.len(), length);

c2r.process(&mut spectrum, &mut outdata).unwrap();
```

### Versions
- 3.0.0: Improved error reporting.
- 2.0.1: Minor bugfix.
- 2.0.0: Update RustFFT to 6.0.0 and num-complex to 0.4.0.
- 1.1.0: Add missing Sync+Send.
- 1.0.0: First version with new api.


### Compatibility

The `realfft` crate requires rustc version 1.37 or newer.

License: MIT
