**[L-1]:** Lacks good layout organization, especially in `LiquidInfrastructureERC20.sol`, which can cause confusion and errors.

**Description:** The contract's functions, modifiers and state variables are spread all over and lack clear and logical documentation or grouping. Hence there is a problem when trying to understand the functionality and structure.

```javascript
   https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L29
```


**Impact:** The omission can cause `increased complexity`, `reduced readability` and `oversight risk`. This means that future teams may need massive resources in terms of `time and effort` to `familiarize with the code`, and might still fail to gain some fundamentals. Auditors might also find it difficult to spot some common bugs, which in the end could be catastrophic to the protocol. 

**Proof of Concept:** Improper organization is evident because state variables and modifiers are scattered without clear context or documentation. Again, comments are sporadic, not fully adhering to the `natspec` configuration strategy. Hence, there can be general confusion about the dependencies and purpose of some functions. 

**Recommended Mitigation:** Try using the `Solidity Smart Contract Layout`. An example layout is shown below;

```javascript
      // Layout of Contract:
      // version
      // imports
      // errors
      // interfaces, libraries, contracts
      // Type declarations
      // State variables
      // Events
      // Modifiers
      // Functions

      // Layout of Functions:
      // constructor
      // receive function (if exists)
      // fallback function (if exists)
      // external
      // public
      // internal
      // Private
      // internal & private view & pure functions
      // external & public view & pure functions
```
