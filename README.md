# EOSIO Weakness Classification Registry
[![CircleCI](https://circleci.com/gh/SmartContractSecurity/SWC-registry/tree/master.svg?style=svg)](https://circleci.com/gh/SmartContractSecurity/SWC-registry/tree/master)
[![Pages](https://img.shields.io/badge/pages-online-blue.svg)](https://smartcontractsecurity.github.io/SWC-registry/)
[![Discord](https://img.shields.io/discord/481002907366588416.svg)](https://discord.gg/qcNvR2r)


The EOSIO smart contract Weakness Classification registry is a classification system for weaknesses and vulnerabilities in EOSIO smart contracts.

It is loosely aligned to the terminologies and structure used in the Common Weakness Classification ([CWE](https://cwe.mitre.org/)) and the Ethereum Smart Contract Weakness Classification ([SWC](https://github.com/SmartContractSecurity/SWC-registry)) Registry whilst paying attention to weakness variants that are specific to EOSIO smart contracts.

## Create a new EOSIO WCR entry

Make sure that there is no matching weakness in the registry. Ideally, also coordinate with the community in [#EOSIO WCR-registry](https://discord.gg/qcNvR2r) to prevent conflicting entries. Create a file with a new EOSIO WCR ID in the [entries](./entries) directory. Use the [template](./entries/template.md) and describe all weakness attributes. 

```
# Title 
Pick a meaningful title.

## Relationships
Link a CWE Base or Class type to the CWS variant. 
e.g.  [CWE-682: Incorrect Calculation](https://cwe.mitre.org/data/definitions/682.html)

## Description 
Describe the nature and potential impact of the weakness on the contract system. 

## Remediation
Describe ways on how to fix the weakness. 

## References 
Link to external references that contain useful additional information on the issue. 

```

## Create a new test case  

Test cases should be as varied as possible and include both simple test cases and real-world samples of vulnerable smart contracts. The test cases are grouped into subdirectories based on a single weakness variant or based on more complex real world contract systems that can contain various weakness variants. A single test case consists of the following structure:

1. A directory that contains all files belonging to a single test case
2. One or multiple source files containing issues. Zero issues is valid as well. These test cases demonstrate how a vulnerable test case can be fixed.
3. A single combined JSON file that contains the compilation result for the main source file and its imports. 
4. The configuration file defining the types and number of weaknesses contained in the contract file

One more thing, make sure the credit the author and mention the source if you don't write the contract sample yourself.

```
/*
 * @source: <link>
 * @author: <name>
 */
```

## Configuration

The configuration contains meta-information about the weaknesses contained in a particular sample. E.g.:

```
1: description: Plain and simple ADD overflow example
2: issues:
3: - id: SWC-101
4:   count: 1
5:   locations:
6:   - bytecode_offsets:
7:       '0x75ad68f906456e1cbfd6190a8f2e2dc5cb2794af4a4929448378642c992e151a': [168]
8:     line_numbers:
9:        overflow_simple_add.sol: [7]
```

- Line 1: `description` provides additional information and context for the test case.
- Line 2: A test case has zero, one or multiple `issues` that are listed in the configuration file.
- Line 3: `id` contains the SWC identifier for the particular weakness. Each weakness is described in a markdown file in the [entries](./entries) directory. 
- Line 4: `count` is the number of times that the weakness occurs in the test case.
- Line 5: `locations` has sub attributes that allow humans and tools to easier identify where issues exists in the test case. 
- Line 6-7: `bytecode_offsets` is a tuple consisting of the keccak256 hash of the runtime or creation byte code and a list of valid offsets. 
- Line 8-9: `line_numbers` is a tuple consisting of the source file and a list of valid line numbers. 

## Contributing

Before you create a PR for the first time make sure you have read:

- the sections [Create a new EOSIO WCR entry](#create-a-new-EOSIO WCR-entry) and [Create a new test case](#create-a-new-test-case).
- read several existing EOSIO WCR definitions and their test cases. 

## Scope of Weaknesses 

EOSIO WCRs should be concerned with weaknesses that can be identified within the code of a smart contract, typically C++. 
Weaknesses in 'smart contract adjacent' code should not be included.

## Contact
This repository is maintained by the [Klevoya](https://klevoya.com) team.