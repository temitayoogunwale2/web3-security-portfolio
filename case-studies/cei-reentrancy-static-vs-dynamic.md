 # Synthetic CEI Reentrancy Research Fixture: Static Heuristics vs. Dynamic EVM Execution
                                                                                            
    ## Overview                                                                             
    This standalone bundle provides a deterministic, local-only research fixture designed to
  evaluate the boundary between static structural code pattern detection and dynamic EVM    
  execution semantics.                                                                      
                                                                                            
    The fixture compares a synthetic vault exhibiting a classic Checks-Effects-Interactions 
  (CEI) ordering violation against dynamic execution behavior in modern Solidity (0.8.20).  
                                                                                            
    > **Scope Notice**: This project is an isolated, synthetic academic fixture. It does not
  connect to external networks, public RPCs, or production protocols, and requires no       
  private keys or mainnet assets.                                                           
                                                                                            
    ---                                                                                     
                                                                                            
    ## Contract Variants                                                                    
                                                                                            
    1. **`src/SyntheticVault.sol` (Non-CEI Target)**                                        
       - Implements a low-level call interaction (`msg.sender.call{value: amount}("")`)     
  *before* updating user balances (`balances[msg.sender] -= amount`).                       
       - Uses default Solidity 0.8.20 checked arithmetic.                                   
       - Purpose: Demonstrates why single-function reentrancy with full balance extraction  
  reverts upon stack unwinding.                                                             
                                                                                            
    2. **`src/SyntheticVaultCEI.sol` (Defensive Baseline)**                                 
       - Implements strict Checks-Effects-Interactions ordering: decrements storage balances
  *prior* to executing the external transfer.                                               
       - Purpose: Demonstrates proactive prevention where reentrant invocations fail        
  immediately at the entry guard (`INSUFFICIENT_FUNDS`).                                    
                                                                                            
    3. **`src/SyntheticVaultUnchecked.sol` (Academic Control)**                             
       - Preserves non-CEI ordering, but executes balance deductions within an `unchecked {
...
  }` block, replicating legacy Solidity (<0.8) wrap-around behavior.                        
       - Purpose: Proves that the unwinding revert in `SyntheticVault.sol` is directly      
  caused by checked arithmetic underflow, and demonstrates successful extraction when       
  underflow checks are disabled.                                                            
                                                                                            
    ---                                                                                     
                                                                                            
    ## Tooling Requirements & Dependencies                                                  
                                                                                            
    - **Foundry / Forge**: Version `1.8.1+`                                                 
    - **Solidity Compiler**: `solc 0.8.20`                                                  
    - **Dependencies**: Bundled locally in `lib/forge-std`. No external package installation,
  git cloning, or network connectivity is required.                                         
                                                                                            
    ---                                                                                     
                                                                                            
    ## Reproduction Commands                                                                
                                                                                            
    Execute the full deterministic test suite:                                              
    ```bash                                                                                 
    forge test -vv                                                                          
                                                                                            
  Inspect the exact call frame unwinding and Panic(0x11) trace:                             
                                                                                            
    forge test --match-test test_03_DemonstrateExactUnderflowMechanism -vvvv                
                                                                                            
  Verify transaction rollback on the checked target:                                        
                                                                                            
    forge test --match-test test_02_ReentrancyAttemptRevertsDueToCheckedArithmeticUnderflow 
  -vvvv                                                                                     
                                                                                            
  Compare trace behavior against the CEI implementation:                                    
                                                                                            
    forge test --match-test test_04_ReentrancyFailsCleanlyOnCEIVault -vvvv                  
                                                                                            
  Verify drain behavior on the unchecked control variant:                                   
                                                                                            
    forge test --match-test test_05_UncheckedArithmeticAllowsSuccessfulDrain -vvvv          
  ──────                                                                                    
  ## Verified Execution Results                                                             
                                                                                            
  All 5 test cases pass deterministically:                                                  
                                                                                            
    Ran 5 tests for test/ReentrancyValidation.t.sol:ReentrancyValidationTest                
    [PASS] test_01_NormalDepositAndWithdrawal() (gas: 136335)                               
    [PASS] test_02_ReentrancyAttemptRevertsDueToCheckedArithmeticUnderflow() (gas: 845464)  
    [PASS] test_03_DemonstrateExactUnderflowMechanism() (gas: 831289)                       
    [PASS] test_04_ReentrancyFailsCleanlyOnCEIVault() (gas: 798083)                         
    [PASS] test_05_UncheckedArithmeticAllowsSuccessfulDrain() (gas: 783406)                 
    Suite result: ok. 5 passed; 0 failed; 0 skipped; finished in 11.57ms                    
                                                                                            
  ### Detailed Scenario Outcomes:                                                           
                                                                                            
  • test_01 (Normal Operations): Verifies 1:1 balance parity on honest deposits and         
  withdrawals.                                                                              
  • test_02 (Underflow Rollback): Confirms that attempting bounded reentrant withdrawal     
  against SyntheticVault triggers a transaction-level revert. Honest user balance (10.0 ETH),
  total assets (10.0 ETH), and attacker balance (0.0 ETH net gain) are completely preserved.
  • test_03 (Unwinding Isolation): Demonstrates that the nested reentrant call temporarily  
  succeeds in setting balances[receiver] to 0, but the outer frame panics with Panic(0x11)  
  when executing 0 - amount, rolling back all state.                                        
  • test_04 (CEI Revert): Demonstrates that SyntheticVaultCEI prevents reentrant entry at   
  the door with INSUFFICIENT_FUNDS.
  • test_05 (Unchecked Extraction): Demonstrates that when underflow checks are bypassed,   
  the attacker extracts 3.0 ETH from a 1.0 ETH deposit, causing a 2.0 ETH loss to honest    
  depositors.
  ──────
  ## Experimental Limits & Technical Framing
  
  ### 1. Static Pattern Score vs. Exploitability
  
  In WRRE's static pattern analysis, SyntheticVault.sol received an internal pattern match  
  score of 0.95 for PAT_REENTRANCY_CEI.
  
  • Important: This 0.95 value is an internal architectural alignment score assigned by the 
  pattern engine upon detecting an external call preceding storage mutations.
  • It is NOT a statistical probability of exploitability, nor does it guarantee economic   
  drain.
  • Static pattern detection flags the syntactic ordering risk; dynamic EVM reproduction is 
  required to establish whether the path can execute without reverting.
  
  ### 2. Why Checked Arithmetic Does NOT Replace CEI
  
  The transaction rollback demonstrated in this test applies only to single-function        
  reentrancy where the post-call mutation attempts to subtract from an already-zeroed       
  balance. Default checked arithmetic does not protect against:
  
  • Cross-Function Reentrancy: If a contract has a secondary function (e.g. transfer, mint, 
  or borrow) that reads the un-updated balance without executing an underflowing subtraction.
  • Alternative Scenarios: Partial-withdrawal and alternative-entry-point scenarios require 
  separate analysis; exploitability depends on the contract's accounting, control flow, and 
  reachable state transitions.
  • Read-Only Reentrancy: If an external protocol or oracle inspects the vault's transiently
  distorted state before the outer call finishes.
  
  The Checks-Effects-Interactions (CEI) pattern and explicit reentrancy mutexes             
  (ReentrancyGuard) remain important security practices; the appropriate protections depend 
  on the contract's design and threat model.
