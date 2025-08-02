To address your request, I’ll provide an example of how **Circom** was used in **Tornado Cash** to create zero-knowledge proofs (specifically zk-SNARKs) for anonymous transactions, explain the mathematical formula behind the circuit, and break it down in a way that’s clear and concise, as you requested. I’ll draw on the provided web results where relevant and ensure the explanation is accessible while covering the technical details.

---

### **Overview of Tornado Cash and Circom**
Tornado Cash is a decentralized cryptocurrency mixer on Ethereum that uses zero-knowledge proofs (zk-SNARKs) to break the link between a depositor’s address and a withdrawer’s address, ensuring privacy. It allows users to deposit a fixed amount (e.g., 1 ETH) into a smart contract, store a commitment in a Merkle tree, and later withdraw to a different address by proving knowledge of a secret without revealing it. Circom, a domain-specific language, is used to define the arithmetic circuits that enforce these proofs.

The core idea is to prove that a withdrawal corresponds to a valid deposit without revealing which deposit, using zk-SNARKs generated from Circom circuits.

---

### **Example: Tornado Cash’s Withdraw Circuit**
The primary circuit in Tornado Cash, often referred to as the `Withdraw.circom` circuit, verifies that a user can withdraw funds by proving:
1. They know a **secret** and **nullifier** that hash to a **commitment** stored in the contract’s Merkle tree.
2. The **nullifier hash** is valid and hasn’t been used before (preventing double-spending).
3. The withdrawal is tied to the recipient’s address without revealing private inputs.

Here’s a simplified version of the `Withdraw.circom` circuit, based on the structure described in the web results and typical zk-SNARK implementations:

```circom
pragma circom 2.0.0;
include "node_modules/circomlib/circuits/mimcsponge.circom";
include "node_modules/circomlib/circuits/merkleTree.circom";

template Withdraw(levels) {
    // Public inputs
    signal input root;                // Merkle tree root
    signal input nullifierHash;       // Hash of the nullifier to prevent double-spending
    signal input recipient;           // Address to receive funds
    signal input relayer;             // Relayer address (optional, for gas payment)
    signal input fee;                 // Fee for relayer
    signal input refund;              // Refund amount (if any)
    
    // Private inputs
    signal input nullifier;           // Secret value to prevent double-spending
    signal input secret;              // Secret value for commitment
    signal input pathElements[levels]; // Merkle proof path
    signal input pathIndices[levels];  // Merkle path indices (0 or 1)
    
    // Compute commitment and nullifier hash
    component commitmentHasher = MiMCSponge(2, 220, 1);
    commitmentHasher.ins[0] <== nullifier;
    commitmentHasher.ins[1] <== secret;
    signal commitment;
    commitment <== commitmentHasher.outs[0];
    
    component nullifierHasher = MiMCSponge(1, 220, 1);
    nullifierHasher.ins[0] <== nullifier;
    nullifierHash === nullifierHasher.outs[0];
    
    // Verify Merkle proof
    component tree = MerkleTreeChecker(levels);
    tree.leaf <== commitment;
    tree.root <== root;
    for (var i = 0; i < levels; i++) {
        tree.pathElements[i] <== pathElements[i];
        tree.pathIndices[i] <== pathIndices[i];
    }
    
    // Constrain public inputs to ensure proof is tied to them
    signal recipientSquare;
    signal relayerSquare;
    signal feeSquare;
    signal refundSquare;
    recipientSquare <== recipient * recipient;
    relayerSquare <== relayer * relayer;
    feeSquare <== fee * fee;
    refundSquare <== refund * refund;
}

component main {public [root, nullifierHash, recipient, relayer, fee, refund]} = Withdraw(20);
```

---

### **Explanation of the Circuit**
This circuit enforces the following logic:
1. **Inputs**:
   - **Public Inputs**: `root` (Merkle tree root), `nullifierHash` (to prevent double-spending), `recipient` (withdrawal address), `relayer`, `fee`, and `refund` (for transaction mechanics).
   - **Private Inputs**: `nullifier` and `secret` (used to compute the commitment), `pathElements` and `pathIndices` (for Merkle proof verification).
   
2. **Commitment Hash**:
   - The circuit uses the `MiMCSponge` hash function from `circomlib` to compute the commitment: `commitment = MiMCSponge(nullifier, secret)`.
   - This commitment is a leaf in the Merkle tree stored on-chain.

3. **Nullifier Hash**:
   - The nullifier hash is computed as `nullifierHash = MiMCSponge(nullifier)`.
   - The circuit enforces `nullifierHash === input nullifierHash` to ensure the provided nullifier hash matches the private `nullifier`.

4. **Merkle Proof Verification**:
   - The `MerkleTreeChecker` component verifies that the `commitment` is a leaf in the Merkle tree with the given `root`, using `pathElements` and `pathIndices`.
   - This proves the deposit is valid without revealing which leaf it is.

5. **Preventing Proof Reuse**:
   - The `recipient`, `relayer`, `fee`, and `refund` are squared (e.g., `recipientSquare <== recipient * recipient`) to bind them to the proof. This ensures a malicious party cannot reuse a proof with a different recipient address.

6. **Constraints**:
   - The circuit generates constraints of the form `A * B + C = 0`. For example:
     - For `nullifierHash === nullifierHasher.outs[0]`, the constraint is `nullifierHash - MiMCSponge(nullifier) = 0`.
     - For the Merkle proof, constraints ensure the hash chain from `commitment` to `root` is valid.

---

### **How Tornado Cash Uses This**
Tornado Cash operates as follows:
1. **Deposit**:
   - A user generates a random `secret` and `nullifier`, computes `commitment = MiMCSponge(nullifier, secret)`, and sends it to the Tornado Cash smart contract with a deposit (e.g., 1 ETH).
   - The contract stores `commitment` in a Merkle tree.

2. **Withdrawal**:
   - To withdraw, the user provides a zk-SNARK proof generated from the `Withdraw` circuit, along with public inputs (`root`, `nullifierHash`, `recipient`, etc.).
   - The circuit proves:
     - The user knows a `secret` and `nullifier` that hash to a `commitment` in the Merkle tree.
     - The `nullifierHash` is valid and hasn’t been used before.
     - The proof is tied to the `recipient` address.
   - The smart contract (generated from the circuit using `snarkjs`) verifies the proof. If valid, it sends the funds to `recipient` and marks `nullifierHash` as used to prevent double-spending.

3. **Privacy**:
   - The zk-SNARK ensures that no one can link the withdrawal to the original deposit, as the `secret` and `nullifier` remain private, and the Merkle proof doesn’t reveal which leaf was used.

---

### **Mathematical Formula Behind the Circuit**
The `Withdraw` circuit’s mathematical structure can be described as a set of constraints over a finite field (typically a prime field used in zk-SNARKs, like the BN128 curve’s scalar field). The key formulas are:

1. **Commitment**:
   \[
   commitment = \text{MiMCSponge}(nullifier, secret)
   \]
   - `MiMCSponge` is a hash function optimized for zk-SNARK circuits, producing a single output from two inputs.

2. **Nullifier Hash**:
   \[
   nullifierHash = \text{MiMCSponge}(nullifier)
   \]
   - The circuit enforces:
     \[
     nullifierHash - \text{MiMCSponge}(nullifier) = 0
     \]

3. **Merkle Proof**:
   - For a Merkle tree of depth \( levels \), the circuit computes the root iteratively:
     \[
     h_0 = commitment
     \]
     \[
     h_{i+1} = \begin{cases} 
     \text{Hash}(h_i, pathElements[i]) & \text{if } pathIndices[i] = 0 \\
     \text{Hash}(pathElements[i], h_i) & \text{if } pathIndices[i] = 1 
     \end{cases}
     \]
     - The final constraint is:
       \[
       root - h_{levels} = 0
       \]
     - `Hash` is typically `MiMCSponge` or another zk-friendly hash function.

4. **Recipient Binding**:
   - To prevent proof reuse, the circuit includes:
     \[
     recipientSquare = recipient^2
     \]
     - This creates a constraint like:
       \[
       recipient^2 - recipientSquare = 0
       \]
     - Similar constraints apply to `relayer`, `fee`, and `refund`.

5. **R1CS Constraints**:
   - Circom compiles the circuit into a Rank-1 Constraint System (R1CS), where all computations are expressed as:
     \[
     A \cdot B + C = 0
     \]
     - For example, for `commitment = MiMCSponge(nullifier, secret)`, the MiMCSponge function is broken down into arithmetic operations, each generating constraints of this form.
     - The exact number of constraints depends on the circuit’s complexity (e.g., MiMCSponge and Merkle tree depth).

---

### **Key Points from Web Results**
- **MiMCSponge Hash**: Tornado Cash uses `MiMCSponge` from `circomlib` for hashing, as it’s efficient for zk-SNARKs (Web ID: 0, 3).
- **Merkle Tree**: The commitment is stored in a Merkle tree, and the circuit verifies the Merkle proof (Web ID: 0, 7).
- **Nullifier**: The `nullifierHash` prevents double-spending by being stored on-chain after a withdrawal (Web ID: 7, 17).
- **Recipient Protection**: Squaring the recipient address ensures the proof is tied to a specific address, preventing reuse (Web ID: 0).
- **Bug Fix**: A notable bug in Tornado Cash was fixed by replacing `=` with `<==` in the circuit, ensuring proper constraint enforcement (Web ID: 10).

---

### **Why This Works for Privacy**
The zk-SNARK proof ensures:
- **Completeness**: If the user knows valid `secret`, `nullifier`, and Merkle proof, the proof verifies.
- **Soundness**: A malicious user cannot forge a proof without knowing the correct inputs.
- **Zero-Knowledge**: The proof reveals nothing about `secret`, `nullifier`, or which Merkle leaf was used, only that the statement is true.

---

### **Conclusion**
Tornado Cash uses Circom to define a `Withdraw` circuit that proves a user’s right to withdraw funds without revealing their deposit. The mathematical formulas involve hashing (`MiMCSponge`), Merkle proof verification, and constraints like `A * B + C = 0` to ensure correctness. The circuit’s design ensures privacy by keeping `secret` and `nullifier` private while binding the proof to public inputs like `recipient`.

If you’d like a deeper dive into a specific part (e.g., MiMCSponge’s math, R1CS conversion, or proof generation with `snarkjs`), or if you want another example circuit, let me know!