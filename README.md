# realfft

Real-to-complex FFT and complex-to-real iFFT based on RustFFT

This library is a wrapper for RustFFT that enables faster computations when the input data is real.
It packs a 2*N long real vector into an N long complex vector, which is transformed using a standard FFT.
It then post-processes the result to give only the first half of the complex spectrum, as an N+1 long complex vector.

The iFFT goes through the same steps backwards, to transform an N+1 long complex spectrum to a 2*N long real result.

Compared to just converting the input to a 2*N long complex vector and using a 2*N long FFT, the the speedup that
can be expected in practice is about a factor 2.

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
use realfft::{ComplexToReal, RealToComplex};
use rustfft::num_complex::Complex;
use rustfft::num_traits::Zero;

// make dummy input vector, spectrum and output vectors
let mut indata = vec![0.0f64; 256];
let mut spectrum: Vec<Complex<f64>> = vec![Complex::zero(); 129];
let mut outdata: Vec<f64> = vec![0.0; 256];

//create an FFT and forward transform the input data
let mut r2c = RealToComplex::<f64>::new(256).unwrap();
r2c.process(&indata, &mut spectrum).unwrap();

// create an iFFT and inverse transform the spectum
let mut c2r = ComplexToReal::<f64>::new(256).unwrap();
c2r.process(&spectrum, &mut outdata).unwrap();
```

### Compatibility

The `realfft` crate requires rustc version 1.34 or newer.

License: MIT
