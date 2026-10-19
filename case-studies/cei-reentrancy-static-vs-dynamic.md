# Case Study: CEI Reentrancy — Static Detection vs. Dynamic Verification

## Objective

Evaluate whether WRRE's static source-analysis pipeline can identify a suspicious Checks-Effects-Interactions (CEI) ordering pattern and how deterministic EVM testing changes the resulting security assessment.

**Scope:** Synthetic Solidity contracts, local Foundry execution, Solidity 0.8.20. No production protocols, public networks, or real assets were involved.

## 1. Detection pipeline

1. **Structural source analysis:** Extract contract state variables, function transitions, external calls, and subsequent state mutations using WRRE's regex-based Solidity source parser.
2. **Pattern classification:** Match extracted features against the `PAT_REENTRANCY_CEI` archetype.
3. **Risk prioritisation:** Route the withdrawal transition to deeper verification.
4. **Dynamic verification:** Use Foundry tests to evaluate execution, rollback, balances, and accounting under controlled conditions.

## 2. Static detection results

For the synthetic vault, WRRE identified a withdrawal sequence in which a low-level ETH transfer occurs before the caller's balance and total-assets accounting are updated.

Reported engine output:

- Pattern: `PAT_REENTRANCY_CEI`
- Internal pattern score: `0.95`
- Risk label: `CRITICAL`
- Withdrawal path: `PRIORITY_DEEP_SOLVE`
- Reported risk score: `1.00`

The `0.95` value is an internal heuristic score, not an empirically measured probability that exploitation will succeed.

## 3. Dynamic verification

The reported isolated Foundry run tested three contract variants:

| Variant | Reported outcome |
|---|---|
| Standard checked arithmetic | Reentrant execution reached a post-call arithmetic underflow; the transaction reverted and changes were rolled back |
| CEI reference implementation | The nested withdrawal failed its balance check |
| Unchecked arithmetic control | The attack extracted 3 ETH after depositing 1 ETH, producing 2 ETH in net profit and a corresponding 2 ETH reduction in vault liquidity |

The report records five passing tests covering normal operations, reentrancy, the underflow trace, the CEI reference, and the unchecked control.

These results describe the specific synthetic fixtures and reported test run. The unchecked control demonstrates why execution semantics and accounting behaviour matter when evaluating the consequences of external-call ordering.

## 4. Security interpretation

The experiment illustrates three separate conclusions:

- **Detection:** WRRE identified a suspicious external-call ordering pattern.
- **Execution:** The reported checked-arithmetic fixture reverted during stack unwinding, while the unchecked control completed the reported drain.
- **Mitigation:** The CEI reference prevented the tested nested withdrawal by updating accounting before the external interaction.

The checked-arithmetic result does not establish that Solidity 0.8+ prevents reentrancy generally. Cross-function, read-only, and other state-dependent reentrancy scenarios require separate analysis. CEI and appropriate reentrancy protections remain important defensive practices.

## 5. Limitations

- The Solidity parser uses heuristic, regex-based structural extraction rather than a demonstrated compiler-generated AST.
- The pattern score is not a statistically validated precision measure.
- The results concern a small, synthetic test fixture and do not establish detection accuracy across a broader contract population.
- Independent reproduction requires the exact Solidity fixtures, Foundry test source, configuration, compiler version, and execution instructions.

## 6. Research conclusion

This experiment supports a layered security workflow: automated structural detection identifies candidate hazards, risk gating prioritises investigation, and deterministic tests examine actual execution behaviour before severity and exploitability are concluded.

WRRE is a research and analysis aid, not a substitute for scope-aware manual review and reproducible verification.

**Disclosure:** Synthetic research only. No confidential client findings or production-target details are included. Test outcomes are reported from the documented local run and should not be interpreted as independently reproduced results.
