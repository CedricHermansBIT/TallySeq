# TallySeq preprint

This directory contains the TallySeq manuscript and the benchmark snapshots used in it.

## Benchmark data in the manuscript

The manuscript uses a repeated Pasilla compatibility/performance matrix, a controlled read-count scaling benchmark, and a large biological input-format benchmark.

The Pasilla benchmark covers 31 scenarios on each of two chromosome 4 datasets, GSM461177 and GSM461178. Each dataset/scenario combination is measured five times for TallySeq and five times for HTSeq 2.1.2 after exact normalized count equality is confirmed. TallySeq uses `--threads 1` and both tools use `-n 1`. On the native BAM reader, `--threads 1` means one additional BGZF decompression worker; the provenance records that setting explicitly.

The scaling benchmark uses real single-end Pasilla alignments and deterministically cycles them to 100,000, 1,000,000, 5,000,000, and 10,000,000 records. It does not alter alignment coordinates, CIGAR strings, MAPQ values, tags, strands, or sequences. Each target and repeat must pass exact count equality before its timing is accepted.

The checked-in manuscript support files are:

- `benchmark_snapshot.tsv`: all 62 real-data dataset/scenario comparisons
- `benchmark_macros.tex`: aggregate real-data values used by the manuscript
- `benchmark_provenance.json`: real-data benchmark settings and system provenance
- `scaling_snapshot.tsv`: the four controlled read-count scaling targets
- `scaling_macros.tex`: scaling values used by the manuscript
- `scaling_provenance.json`: scaling settings, system provenance, and fitted coefficients
- `large_biological_snapshot.tsv`: median SAM/BAM/CRAM results for SRR5724993
- `large_biological_macros.tex`: large-dataset values used by the manuscript
- `large_biological_provenance.json`: raw large-dataset measurements, tool versions, input metadata, and counting options

## Rebuilding benchmark values

Run the real-data benchmark:

```bash
python benchmarks/benchmark_real_data.py \
  --build \
  --profile full \
  --dataset GSM461177 \
  --dataset GSM461178 \
  --rust-threads 1 \
  --nprocesses 1 \
  --repeats 5
```

Run the controlled scaling benchmark:

```bash
python benchmarks/benchmark_scaling.py \
  --rust-bin target/release/tallyseq \
  --dataset GSM461177 \
  --rust-threads 1 \
  --nprocesses 1 \
  --repeats 5 \
  --records 100000 \
  --records 1000000 \
  --records 5000000 \
  --records 10000000
```

Regenerate all manuscript benchmark support files with:

```bash
python paper/update_benchmark.py \
  benchmarks/results/real_data_benchmark.json \
  --scaling-report benchmarks/results/scaling_benchmark.json
```

## Compile

With a standard TeX installation:

```bash
cd paper
latexmk -pdf main.tex
```

The manuscript uses standard packages plus `siunitx`, `natbib`, `authblk`, `booktabs`, `microtype`, and `hyperref`.


## Large biological benchmark

The large biological benchmark uses local SRR5724993 files and is not downloaded or stored in this repository. The source BAM contains 58,663,336 mapped primary single-end alignments. SAM and CRAM representations were generated from that BAM and verified to contain the same number of records.

For the strict BAM comparison, TallySeq uses `--threads 0`, which disables additional BGZF decompression workers. The SAM reader does not use this BAM worker setting. CRAM is measured with no additional HTSlib decoding threads and includes TallySeq's CRAM-to-temporary-BAM compatibility conversion.

The checked-in `large_biological_provenance.json` contains the raw timing/RSS measurements and exact counting settings, but not the large alignment or annotation files.
