# PokerBot - Heads-Up Poker GTO Solver

A high-performance C++ implementation of a Counterfactual Regret Minimization (CFR) solver for heads-up No-Limit Texas Hold'em poker. This solver computes Game Theory Optimal (GTO) strategies using parallelized CFR algorithms.

## Overview

PokerBot is an interactive Texas Hold'em heads-up bot that applies a fully functional process, leverages Game Theory Optimal (GTO) ranges, and a Counterfactual Regret Minimization (CFR) algorithm for frequency-based and exploitative decision-making. Simulates heads-up play with advanced GTO raise sizings and can be used on C++ compilers. Built upon existing highly hand optimized C++ poker hand evaluation library. The core libraries of pokerstove are being open sourced. The project is currently in the process of reviewing and publishing the code. As code is reviewed and code sanitized further commits will be added.

This project implements a complete poker GTO solver capable of:
- Computing Nash equilibrium strategies for heads-up poker scenarios
- Training using Counterfactual Regret Minimization (CFR/CFR+)
- Managing preflop hand ranges and reach probabilities
- Evaluating exploitability to measure GTO convergence
- Parallel computation using Intel Threading Building Blocks (TBB)

## Features

### CFR Algorithm Implementation
- **Counterfactual Regret Minimization**: Industry-standard algorithm for solving imperfect information games
- **Parallel Processing**: Utilizes Intel TBB for efficient multi-threaded computation
- **Exploitability Tracking**: Measures how close the solution is to true GTO
- **Best Response Calculation**: Computes optimal counter-strategies
- **Frequency-Based Decision Making**: Uses CFR to determine optimal action frequencies

### Game Tree Construction
- **Flexible Street Support**: Supports flop, turn, and river play
- **Configurable Bet Sizing**: Customizable bet sizes and raise amounts
- **Action Abstraction**: Efficient action space representation
- **Stack Depth Handling**: Accounts for varying stack sizes and pot odds

### Range Management
- **Preflop Ranges**: Supports all 169 unique starting hand combinations
- **Reach Probability Tracking**: Maintains probability distributions across game tree
- **Board Texture Analysis**: Adjusts ranges based on board cards
- **Hand Strength Evaluation**: Uses highly optimized hand evaluation libraries

## Project Structure

```
PokerBot/
├── src/               # Source files
│   ├── Action.cpp/h           # Action representation
│   ├── ActionNode.cpp/h       # Player decision nodes
│   ├── BestResponse.cpp/h     # Best response calculator
│   ├── BestResponseTask.cpp/h # Parallel best response
│   ├── BetSettings.h          # Bet size configuration
│   ├── CfrTask.cpp/h          # CFR algorithm tasks
│   ├── ChanceNode.cpp/h       # Board card dealing nodes
│   ├── GameTree.cpp/h         # Game tree builder
│   ├── Hand.cpp/h             # Hand representation
│   ├── HandEvaluator.cpp/h    # Hand strength evaluation
│   ├── PlayerState.cpp/h      # Player stack/position state
│   ├── RangeManager.cpp/h     # Range and probability management
│   ├── ShowdownTask.cpp/h     # Showdown computation
│   ├── State.cpp/h            # Game state tracking
│   ├── TerminalNode.cpp/h     # Terminal game states
│   ├── Trainer.cpp/h          # Main training loop
│   ├── TreeBuildSettings.cpp/h # Tree configuration
│   ├── card_utility.cpp/h     # Card utility functions
│   └── program.cpp            # Main entry point with examples
├── data/
│   └── HandRanks.dat          # TwoPlusTwo hand evaluator data
├── README.md
├── CMakeLists.txt
├── LICENSE
└── .gitignore
```

## Requirements

### Dependencies
- C++11 or later
- Intel Threading Building Blocks (TBB)
- CMake 3.10+ (for building)

### Operating System
- macOS, Linux, or Windows

## Installation

### 1. Install Dependencies

**macOS:**
```bash
brew install tbb cmake
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt-get install libtbb-dev cmake
```

**Windows:**
- Install TBB from Intel or vcpkg
- Install CMake from cmake.org

### 2. Download HandRanks.dat

The TwoPlusTwo hand evaluator requires a data file:
```bash
cd data/
curl -O https://raw.githubusercontent.com/christophschmalhofer/poker/master/XPokerEval/XPokerEval.TwoPlusTwo/HandRanks.dat
```

### 3. Build the Project

```bash
mkdir build
cd build
cmake ..
make
```

## Usage

### Running Example Scenarios

The `program.cpp` file contains several example scenarios:

```bash
./PokerBot
```

### Configuring a Custom Scenario

```cpp
// Define starting hand ranges for both players
string p1StartingHands = "AA,KK,QQ,JJ,TT,99,88,77,66,55,44,33,22,AK,AQ,AJ";
string p2StartingHands = "AA,KK,QQ,JJ,TT,99,88,77,66,55,44,33,22,AK,AQ,AJ";

// Set initial game state
int inPositionPlayerId = 2;  // Player 2 has position
Street initialStreet = Street::TURN;
uint8_t initialBoard[5] = {
    card_from_string("Kd"),
    card_from_string("Jd"),
    card_from_string("Td"),
    card_from_string("5s"),
    52  // No river card yet
};

int initialPotSize = 100;
int startingStackSize = 1000;

// Configure bet sizes
unique_ptr<BetSettings> p1BetSettings = make_unique<BetSettings>();
p1BetSettings->turnBetSizes.push_back(0.5f);  // 50% pot bet
p1BetSettings->turnBetSizes.push_back(1.0f);  // Pot-sized bet

// Build game tree
unique_ptr<TreeBuildSettings> treeBuildSettings = make_unique<TreeBuildSettings>(
    rangeManager, inPositionPlayerId, initialStreet, initialBoard,
    initialPotSize, startingStackSize,
    move(p1BetSettings), move(p2BetSettings),
    minimumBetSize, allinThreshold
);

unique_ptr<GameTree> gameTree = make_unique<GameTree>(move(treeBuildSettings));
unique_ptr<Node> root = gameTree->build();

// Train the solver
unique_ptr<Trainer> trainer = make_unique<Trainer>(
    rangeManager, initialBoard, initialPotSize, inPositionPlayerId
);
trainer->train(root.get(), 1000);  // 1000 CFR iterations
```

### Understanding Output

During training, the solver outputs:
- **Initial Exploitability**: How exploitable the strategy is before training
- **Iteration Progress**: Updates every 25 iterations
- **Final Exploitability**: Convergence toward GTO (lower is better)
- **Training Time**: Computation time for iterations

Example output:
```
Player 1 Exploitability: 45.23
Player 2 Exploitability: 42.17
Total Exploitability: 87.40

25 cfr iterations took: 2.34s
Player 1 Exploitability: 12.45
Player 2 Exploitability: 11.89
Total Exploitability: 24.34
```

## Algorithm Explanation

### Counterfactual Regret Minimization (CFR)

CFR is an iterative algorithm that:

1. **Traverses the game tree** for each player
2. **Calculates regrets** for not taking each action
3. **Updates strategies** based on cumulative regrets
4. **Converges to Nash equilibrium** over many iterations

The algorithm maintains:
- **Regret values**: How much a player regrets not taking an action
- **Strategy sum**: Cumulative strategy weights over all iterations
- **Current strategy**: Probability distribution over actions

### Reach Probabilities

Reach probabilities track the likelihood of reaching each game state given:
- Player's starting range
- Actions taken by both players
- Board cards dealt

## Performance Optimization

This implementation is optimized for speed:
- **2-3x faster** than reference JavaScript implementation
- **Parallel CFR computation** using Intel TBB
- **Efficient hand evaluation** with highly optimized C++ libraries
- **Smart memory management** with move semantics

## Example Scenarios

The project includes several pre-configured scenarios:

### 1. Turn Play (Deep Stacks)
- 100bb effective stack
- Multiple bet sizes
- Tests value betting and bluffing frequencies

### 2. River Play
- All-in or fold decisions
- Polarized ranges
- Tests GTO calling frequencies

### 3. Flop Play
- Full game tree from flop
- Multiple streets
- Comprehensive strategy computation

## Contributing

Contributions are welcome! Areas for improvement:
- Additional betting abstractions
- CFR+ and other CFR variants
- Preflop solver integration
- Strategy visualization tools
- Performance profiling

## Libraries Used

**PokerStove** by Andrew Prock - Highly optimized C++ poker hand evaluation library

## Credits

Based on the algorithms from [discounted-cfr-poker-solver](https://github.com/Fossana/discounted-cfr-poker-solver) with significant C++ optimizations.

Copyright (c) 2012, Andrew C. Prock. All rights reserved.

## License

### PokerBot

MIT License - See LICENSE file for details

### PokerStove

Copyright (c) 2012, Andrew C. Prock. All rights reserved.

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

- Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.

- Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

- Neither the name of the Andrew C. Prock nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

## References

- [An Introduction to Counterfactual Regret Minimization](http://modelai.gettysburg.edu/2013/cfr/cfr.pdf)
- [Modern Poker Theory](https://www.modernpokertheory.com/) by Michael Acevedo
- [Mathematics of Poker](https://www.amazon.com/Mathematics-Poker-Bill-Chen/dp/1886070253) by Bill Chen and Jerrod Ankenman

## Acknowledgments

This project was created for educational purposes to understand GTO poker strategy and imperfect information game solving.
