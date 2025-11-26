# Intenus: Verifiable Intent Solver Ranking on Sui

Intenus is a **trustless intent solver ranking system** powered by Trusted Execution Environments (TEEs) on Sui blockchain. It enables transparent, verifiable, and manipulation-resistant ranking of solver solutions for intent-based transactions.

## Overview

Intenus combines AI-powered intent classification with TEE-secured ranking computation to provide:

- **Verifiable Rankings**: All ranking computations are performed in AWS Nitro Enclaves with attestation verification on-chain
- **AI Intent Classification**: ML-powered intent analysis determines optimal ranking strategy and weights
- **Multi-Strategy Ranking**: Supports diverse ranking strategies tailored to different intent types (swap, liquidity, transfer, etc.)
- **Off-Chain Efficiency**: Complex ranking computations happen off-chain while maintaining on-chain verifiability
- **Reproducible Builds**: Anyone can verify that the enclave code matches the published source

## Architecture

The system consists of three main components:

1. **AI Intent Classifier** (`ai-main/`): FastAPI service that analyzes user intents and predicts optimal ranking strategies and weights
2. **TEE Ranking Service** (`src/nautilus-server/`): Rust-based enclave service that performs secure ranking computations
3. **On-Chain Verification** (`move/intenus-ranking/`): Sui Move smart contracts that verify enclave attestations and ranking results

## How It Works

1. **Intent Submission**: Users submit intents with multiple solver solutions
2. **AI Classification**: The intent classifier analyzes the request and determines the optimal ranking strategy
3. **Secure Ranking**: The enclave ranks solutions using the determined strategy within a TEE
4. **Signature Generation**: The enclave signs the ranking result with its ephemeral key
5. **On-Chain Verification**: The Sui contract verifies the signature and stores the verified ranking
6. **Execution**: The best solution is executed on-chain with cryptographic proof of fair selection

## Example I/O
### Input:
```json
{
  "payload": {
    "intent_id": "intent_test_123",
    "intent_classification": {
      "primary_category": "swap",
      "primary_category_confidence": 0.95,
      "detected_priority": "cost",
      "detected_priority_confidence": 0.9,
      "complexity_level": "simple",
      "complexity_level_confidence": 0.92,
      "risk_level": "low",
      "risk_level_confidence": 0.88,
      "strategy": "Speed-Priority",
      "strategy_confidence": 0.93,
      "weights": {
        "gt_weight_surplus_usd": 0.3,
        "gt_weight_surplus_percentage": 0.0,
        "gt_weight_gas_cost": 0.25,
        "gt_weight_protocol_fees": 0.0,
        "gt_weight_total_hops": 0.0,
        "gt_weight_protocols_count": 0.0,
        "gt_weight_estimated_execution_time": 0.25,
        "gt_weight_solver_reputation_score": 0.2,
        "gt_weight_solver_success_rate": 0.0
      }
    },
    "passed_solution_ids": ["sol_1", "sol_2", "sol_3"],
    "failed_solution_ids": [],
    "feature_vectors": [
      {
        "solution_id": "sol_1",
        "features": {
          "surplus_usd": 100.0,
          "surplus_percentage": 2.0,
          "gas_cost": 10.0,
          "protocol_fees": 5.0,
          "total_cost": 15.0,
          "total_hops": 2,
          "protocols_count": 1,
          "estimated_execution_time": 1000.0,
          "solver_reputation_score": 0.95,
          "solver_success_rate": 0.98
        }
      },
      {
        "solution_id": "sol_2",
        "features": {
          "surplus_usd": 95.0,
          "surplus_percentage": 1.8,
          "gas_cost": 8.0,
          "protocol_fees": 4.0,
          "total_cost": 12.0,
          "total_hops": 3,
          "protocols_count": 2,
          "estimated_execution_time": 1500.0,
          "solver_reputation_score": 0.88,
          "solver_success_rate": 0.92
        }
      },
      {
        "solution_id": "sol_3",
        "features": {
          "surplus_usd": 105.0,
          "surplus_percentage": 2.2,
          "gas_cost": 12.0,
          "protocol_fees": 6.0,
          "total_cost": 18.0,
          "total_hops": 1,
          "protocols_count": 1,
          "estimated_execution_time": 800.0,
          "solver_reputation_score": 0.92,
          "solver_success_rate": 0.96
        }
      }
    ],
    "dry_run_results": [],
    "stats": {
      "total_submitted": 3,
      "passed": 3,
      "failed": 0,
      "dry_run_executed": 3,
      "dry_run_successful": 3
    },
    "processed_at": 1744684007462
  }
}
```

### Output:
```json
{
    "response": {
        "intent": 0,
        "timestamp_ms": 1764119122843,
        "data": {
            "intent_id": "intent_test_123",
            "ranked_solutions": [
                {
                    "rank": 1,
                    "score": 3152,
                    "solution_id": "sol_3",
                    "solver_address": "unknown",
                    "transaction_bytes": "0x00",
                    "score_breakdown": {
                        "surplus_score": 1740,
                        "cost_score": 9040,
                        "speed_score": 8760,
                        "reputation_score": 9360
                    },
                    "reasoning": {
                        "primary_reason": "Highly cost-efficient solution minimizing fees",
                        "secondary_reasons": [
                            "Simple routing path",
                            "Single protocol usage reduces risk",
                            "Above 1% surplus generation",
                            "Highly reliable solver"
                        ],
                        "risk_assessment": "low",
                        "confidence_level": 9150
                    },
                    "personalization_applied": false,
                    "warnings": [],
                    "expires_at": 1764119422843
                },
                {
                    "rank": 2,
                    "score": 3003,
                    "solution_id": "sol_1",
                    "solver_address": "unknown",
                    "transaction_bytes": "0x00",
                    "score_breakdown": {
                        "surplus_score": 1600,
                        "cost_score": 9200,
                        "speed_score": 8300,
                        "reputation_score": 9620
                    },
                    "reasoning": {
                        "primary_reason": "Highly cost-efficient solution minimizing fees",
                        "secondary_reasons": [
                            "Simple routing path",
                            "Single protocol usage reduces risk",
                            "Above 1% surplus generation",
                            "Highly reliable solver"
                        ],
                        "risk_assessment": "low",
                        "confidence_level": 9150
                    },
                    "personalization_applied": false,
                    "warnings": [],
                    "expires_at": 1764119422843
                },
                {
                    "rank": 3,
                    "score": 2853,
                    "solution_id": "sol_2",
                    "solver_address": "unknown",
                    "transaction_bytes": "0x00",
                    "score_breakdown": {
                        "surplus_score": 1460,
                        "cost_score": 9360,
                        "speed_score": 7150,
                        "reputation_score": 8960
                    },
                    "reasoning": {
                        "primary_reason": "Highly cost-efficient solution minimizing fees",
                        "secondary_reasons": [
                            "Above 1% surplus generation"
                        ],
                        "risk_assessment": "low",
                        "confidence_level": 9150
                    },
                    "personalization_applied": false,
                    "warnings": [],
                    "expires_at": 1764119422843
                }
            ],
            "best_solution": {
                "rank": 1,
                "score": 3152,
                "solution_id": "sol_3",
                "solver_address": "unknown",
                "transaction_bytes": "0x00",
                "score_breakdown": {
                    "surplus_score": 1740,
                    "cost_score": 9040,
                    "speed_score": 8760,
                    "reputation_score": 9360
                },
                "reasoning": {
                    "primary_reason": "Highly cost-efficient solution minimizing fees",
                    "secondary_reasons": [
                        "Simple routing path",
                        "Single protocol usage reduces risk",
                        "Above 1% surplus generation",
                        "Highly reliable solver"
                    ],
                    "risk_assessment": "low",
                    "confidence_level": 9150
                },
                "personalization_applied": false,
                "warnings": [],
                "expires_at": 1764119422843
            },
            "metadata": {
                "total_solutions": 3,
                "average_score": 3002,
                "strategy": "Speed-Priority",
                "intent_category": "swap"
            },
            "ranked_at": 1764119122843,
            "expires_at": 1764119422843
        }
    },
    "signature": "81ad06ea1f797a1bfa5e58a5f19c38dab6b1e5c2239c488f91061a470d8d36a07c3dc7f99d7dd15e3abda1fefdeeca366aad36836a360c647f99bc3830dc830a"
}
```


## Key Features

### Ranking Strategies
- **Surplus Maximization**: Optimizes for user value (best price)
- **Gas Optimization**: Minimizes transaction costs
- **Speed Prioritization**: Fastest execution time
- **Balanced Approach**: Multi-objective optimization
- **Risk-Adjusted**: Considers solver reputation and historical performance

### Security Model
- Built on Nautilus framework for verifiable off-chain computation
- AWS Nitro Enclaves provide hardware-level isolation
- Reproducible builds enable independent verification
- On-chain attestation verification ensures enclave integrity
- Ephemeral key signing prevents replay attacks

## Repository Structure

```
├── ai-main/              # Intent classifier ML service
├── move/                 # Sui Move smart contracts
│   ├── enclave/         # Base enclave module
│   └── intenus-ranking/ # Ranking verification contracts
├── src/
│   ├── nautilus-server/ # TEE ranking service
│   ├── aws/            # AWS integration
│   └── system/         # System utilities
└── scripts/            # Build and deployment scripts
```
