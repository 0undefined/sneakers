# Power Sneakers

As we all know, the power sneakers are the sneakers the BlueHedgehog wears.
This repository provides the `sneaker` utility, a simple bash script that runs
futhark benchmarks on different backends using `futhark bench` and collects the
results into a neat csv format, for easy plotting of performance.

It currently _DOES NOT_ support multiple entries or random inputs.

# Dependencies

* Bash
* sed, awk
* Futhark


# Usage

```txt
Options:
  -k ARG              The benchmark entrys ith input argument to use for 'n' in
                      the output csv file.
  -b BACKEND(S)       A comma seperated list of backends to run the benchmark
                      on. Default is: "c,ispc,multicore,opencl"
  -o CSV              Name of the output csv file, printed to stdout otherwise.
  --skip-benchmarks   Skip running the actual benchmarks, useful if they have
                      already been run, but not formatted
  --skip-csv          Skip printing/writing the csv output.
  --quiet             Do not print anything.
  --verbose           Do not suppress output when piped.
  --help              Show this help message and exit.

  --                  all arguments after \"--\" is passed to "futhark-bench"
```

## Examples

## Simple program

suppose we have an entry specified as
```futhark
-- ==
-- input { 100i64 }
-- input { 150i64 }
-- input { 200i64 }
entry bench_one (n: i64) i64 =
  ...
```
Then we can benchmark it running the following:

    sneaker main.fut -- -e bench_one -r 250

This will tell futhark bench to use the bench_one entry and repeat the benchmark
250 times.

An example output could be
```
n,c,ispc,multicore,opencl
100,2,42,28,264
150,4,50,32,247
200,10,45,35,281
```


## Multiple arguments

Say we have a slightly more complicated benchmark, defined as follows:
```futhark
-- ==
-- input { 1.2f32 100i64 }
-- input { 2.0f32 150i64 }
-- input { 3.1f32 200i64 }
entry bench_two (d: f32) (n: i64) i64 =
  ...
```
We can then pick out the second input argument as `n` in the csv output using
`-k 1` (the arguments are zero indexed)

  sneaker -k 1 main.fut

## Specifying backends

If one just wants to compare the c and cuda backends performance with a single
given program, then one can run

  sneaker -b c,cuda main.fut


## Merging benchmarks

Say, you have two implementations of the same thing, and want to merge the two
benchmarks into the same csv file. Then, given they get the same input, then you
can run them seperately with

```txt
sneaker --skip-csv -b cuda prog.fut -- -e impl_one
mv result-cuda.bench result-imp1.bench
sneaker --skip-csv -b cuda prog.fut -- -e impl_two
mv result-cuda.bench result-imp2.bench
```
and merge them using
```
sneaker --skip-benchmarks -b imp1,imp2 prog.fut
```

This will simply match each of the new result-imp1.fut and result-imp2.fut files
and merge them into one tidy csv.
