# Data Directory

This directory contains data files required for the PokerBot solver.

## HandRanks.dat

The `HandRanks.dat` file is required for fast poker hand evaluation using the TwoPlusTwo evaluator.

### Download Instructions

```bash
cd data/
curl -O https://raw.githubusercontent.com/christophschmalhofer/poker/master/XPokerEval/XPokerEval.TwoPlusTwo/HandRanks.dat
```

Or download manually from:
https://github.com/christophschmalhofer/poker/blob/master/XPokerEval/XPokerEval.TwoPlusTwo/HandRanks.dat

### File Information

- **File Name**: HandRanks.dat
- **Size**: ~130 MB
- **Purpose**: Lookup table for fast poker hand strength evaluation
- **Algorithm**: TwoPlusTwo hand evaluator

### Note

This file is **not** included in the repository due to its size. You must download it separately before running the solver.

## Additional Data Files

You can add custom range files, strategy outputs, or training results to this directory.

### Suggested Structure

```
data/
├── HandRanks.dat           # Required evaluator data
├── ranges/                 # Custom preflop range files
├── strategies/             # Saved GTO strategies
└── results/               # Training results and logs
```
