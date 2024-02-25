## Overview of the Open Dollar Audit

Open Dollar is a DeFi lending protocol designed for liquidity and stability. Users can lock Liquid Staking Tokens (LSTs) and assets into Collateralized Debt Positions (CDPs) through NFT Vaults, allowing them to borrow the &#36;OD stablecoin. Unlike other stablecoins, &#36;OD adjusts its redemption rate based on market volatility, enhancing flexibility and minimizing governance. Open Dollar aims to reduce LST volatility, boost token liquidity, and offers a unique tradable vaults system. It's a floating &#36;1.00 pegged stablecoin backed by LSTs, primarily for Arbitrum, and employs the GEB framework for CDPs. This protocol enables borrowing against staked assets, earning staking rewards, and liquidity through Non-Fungible Vaults (NFVs).

**The key contracts of Open Dollar protocol for this Audit are**:

These contracts are central to the functionality, security, and governance of the Open Dollar protocol. Focusing on them first will provide a solid foundation for understanding the protocol's operation and how it manages user assets and stability.

- **ODGovernor.sol**:
  ODGovernor contract manages the governance of the protocol, enabling modifications to key parameters. Understanding how governance functions is crucial for ensuring the protocol's stability and adaptability.

- **AccountingEngine.sol**:
  The AccountingEngine contract handles the surplus, debt, and auctions. It's essential to comprehend how the protocol manages these aspects as they directly impact the stability and liquidity of the platform.

- **Vault721.sol**:
  This contract serves as the Proxy Registry and Proxy Factory and is responsible for managing safe ownership, transfers, and approvals. Understanding how safes are managed is vital to ensure the security of users' assets.

- **ODProxy.sol**:
  ODProxy is critical for ensuring the safe and secure transfer of assets within the protocol. It's important to understand its functionality and restrictions.

- **ODSafeManager.sol**:
  As it manages safe movement and creation, it's essential to know how this contract operates, especially in terms of the interaction with Vault721 and safe management.

### System Overview

### Scope

- **AccountingEngine.sol**: AccountingEngine contract in the Open Dollar protocol manages the protocol's surplus and debt. It facilitates surplus and debt auctions, tracks debt queues, and enables settlement. Additionally, it can initiate surplus auctions, transfer surplus to designated receivers, and be disabled to handle post-settlement debt and surplus management.

- **CamelotRelayer.sol**: The CamelotRelayer contract in the Open Dollar protocol bridges data from a CamelotRelayer TWAP pool to a standard IBaseOracle format. It converts the price into a 18-decimal format. The contract is initialized with base and quote tokens, symbol, and other parameters. It offers functions to retrieve results from the CamelotRelayer pool, with checks for data validity. This adaptation is vital for integrating price data within the Open Dollar protocol.

- **CamelotRelayerFactory.sol**: The CamelotRelayerFactory contract in the Open Dollar protocol deploys CamelotRelayers, which fetch and transform price data. It's initialized with authorization controls and provides a method to create new CamelotRelayers with specific token pairs and periods. The contract maintains a list of deployed CamelotRelayers, essential for obtaining reliable price data within the Open Dollar system.

- **CamelotRelayerChild.sol**: The CamelotRelayerChild contract inherits all the functionality of the "CamelotRelayer" contract and is specifically designed to be deployed by the factory. It serves as a CamelotRelayer within the Open Dollar protocol, providing price data transformation and compatibility features for base and quote tokens with specified quote periods.

- **UniV3Relayer.sol**: The UniV3Relayer contract in the Open Dollar protocol fetches and transforms price data from a UniswapV3Pool into a standard IBaseOracle feed. It's configured with specific base and quote tokens, along with a fee tier and quote period for obtaining reliable price data. The contract ensures that the quote result is expressed in an 18 decimal format, making it compatible with the Open Dollar ecosystem.

- **ODGovernor.sol**: The ODGovernor contract in the Open Dollar protocol is a comprehensive governance contract that incorporates various modules and extensions for decentralized decision-making. It manages voting and proposals, configures parameters like voting delay and quorum, and integrates with the Open Dollar token and a timelock contract to ensure secure and effective governance processes.

- **ODProxy.sol**: The ODProxy contract is a simple proxy contract designed for use in the Open Dollar protocol. It allows the owner to execute functions on another contract (the target) via delegatecall. The contract enforces that only the owner can invoke these functions. It includes error handling for cases when the target address is required, the target call fails, or the sender is not the owner.

- **Vault721.sol**: The Vault721 contract is used in the Open Dollar protocol (Version 1.5.5) to manage tradable Vaults as ERC-721 tokens. It involves governance, initialization, proxy creation, and management. Users can interact with their Vaults and trade ownership of these assets through this contract.

- **SAFEHandler.sol**: The SAFEHandler contract is used to create a unique safe handler address for each user's SAFE within the Open Dollar protocol. It is deployed when a new SAFE is created and grants permissions to the SAFE manager to modify the SAFE associated with this contract's address.

- **ODSafeManager.sol**: The ODSafeManager contract serves as an interface to the SAFEEngine, facilitating the management of SAFEs in the Open Dollar protocol. It provides functions to open, modify, and transfer SAFEs, as well as control access permissions. The contract also interacts with the Vault721 contract for NFT management.

- **BasicActions.sol**: The BasicActions contract in the Open Dollar protocol facilitates the management of SAFEs. It allows users to open new SAFEs, generate and repay debt, lock and free token collateral, and manage collateralization ratios. These actions are crucial for maintaining the stability and usability of the protocol's stablecoin system.

### Privileged Roles

Some privileged roles exercise powers over the Controller contract:

- **Owner**

```solidity
  modifier onlyOwner() {
    if (msg.sender != OWNER) revert OnlyOwner();
    _;
  }
```

- **Control Access For DAO Governor**

```solidity
  modifier onlyGovernor() {
    if (msg.sender != governor) revert NotGovernor();
    _;
  }
```

- **Owner Of The Safe**

```solidity
  modifier safeAllowed(uint256 _safe) {
    address _owner = _safeData[_safe].owner;
    if (msg.sender != _owner && safeCan[_owner][_safe][msg.sender] == 0) revert SafeNotAllowed();
    _;
  }
```

- **Safe Handler**

```solidity
  modifier handlerAllowed(address _handler) {
    if (msg.sender != _handler && handlerCan[_handler][msg.sender] == 0) revert HandlerNotAllowed();
    _;
  }
```

### Approach Taken-in Evaluating The Open Dollar Protocol

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contract Overview**:

    I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality.
    The main goal was to take a close look at the important contracts and how they work together in the Open Dollar Protocol.

    **Main Contracts I Looked At**

                ODGovernor.sol
                Vault721.sol
                ODSafeManager.sol
                ODProxy.sol
                SAFEHandler.sol
                AccountingEngine.sol
                BasicActions.sol
                CamelotRelayer.sol

    I started my analysis by examining the ODGovernor.sol contract. Audit the governance contract first as it has a central role in modifying protocol parameters, enabling modifications to protocol parameters and security settings. Its functions for proposal creation, voting, and quorum verification must be meticulously audited to ensure secure and accurate governance operations. Particular attention should be paid to timelock control mechanisms, role-based access controls, and protection against vulnerabilities.
**Comments and Documentation**: While the codebase contains comments, improving the quality of comments and documentation can enhance code readability. Consider using more explanatory comments to clarify the purpose and functionality of each section of the code. This will make it easier for other developers and auditor to understand and maintain the code.

**Decimals Handling**: When working with decimals, ensure that all conversions are handled accurately.For-Example The multiplier calculation in CamelotRelayer.sol could potentially benefit from additional comments and explanation for clarity.

**Formal Verification**: Consider a professional formal verification of the contract to identify and mitigate any security risks.

**Consolidation of Related Functions**:For-Exampe: Some functions like `_generateDebt` and `_repayDebt` in BasicActions.sol share common functionality, and combining them into a single function with parameters to specify the operation could reduce code duplication.

### Codebase Quality

Overall, I consider the quality of the Open Dollar codebase to be excellent. The code appears to be very mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Maintainability and Reliability** | The codebase demonstrates moderate maintainability with well-structured functions and comments, promoting readability. It exhibits reliability through defensive programming practices, parameter validation, and handling external calls safely. The use of internal functions for related operations enhances code modularity, reducing duplication. Libraries improve reliability by minimizing arithmetic errors. Adherence to standard conventions and practices contributes to overall code quality. However, full reliability depends on external contract implementations like openzeppelin, uniswap.                                                                       |
| **Code Comments**                        | The contracts have comments that are used to explain the purpose and functionality of different parts of the contracts, making it easier for developers to understand and maintain the code. The comments provide descriptions of methods, variables, and the overall structure of the contract.For-Exmaple: The code comments in the "CamelotRelayerChild" contract provide essential information. The imported interfaces and contracts are declared, and the contract's title and purpose are described.                                                                                                                                                                         |
| **Documentation**                        | The documentation of the Open Dollar project is quite comprehensive and detailed, providing a solid overview of how Open Dollar is structured and how its various aspects function. However, we have noticed that there is room for additional details, such as diagrams, to gain a deeper understanding of how different contracts interact and the functions they implement. With considerable enthusiasm. We are confident that these diagrams will bring significant value to the protocol as they can be seamlessly integrated into the existing documentation, enriching it and providing a more comprehensive and detailed understanding for users, developers and auditors. |
| **Testing**                              | The audit scope of the contracts to be audited is 95% and it should be aimed to be 100%.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| **Code Structure and Formatting**        | The codebase contracts are well-structured and formatted. It inherits from multiple components, adhering for-example OpenZeppelin governance standards in some contracts. The constructors initializes the contract with parameters and inherited components. Override functions for each component are provided with accompanying comments explaining their purpose. The code is organized, making it readable and maintainable.                                                                                                                                                                                                                                                   |

### Systemic & Centralization Risks

The analysis provided highlights several significant systemic and centralization risks present in the Open Dollar protocol. These risks encompass concentration risk in ODGovernor, BasicActions, AccountingEngine risk and more, third-party dependency risk, and centralization risks arising from the existence of an “owner” role in specific contracts. However, the documentation lacks clarity on whether this address represents an externally owned account (EOA) or a contract, warranting the need for clarification. Additionally, the absence of fuzzing and invariant tests could also pose risks to the protocol’s security.

Here's an analysis of potential systemic and centralization risks in the provided contracts:

### Systemic Risks:

1. **No having, fuzzing and invariant tests could open the door to future vulnerabilities**.

2. The AccountingEngine contract manages debt and surplus within the system, which can be a systemic risk if not executed correctly. Mishandling debt or surplus could lead to imbalances and impact the stability of the entire protocol, and this contract allows parameters to be modified. Inappropriately setting these parameters can introduce systemic risks. For example, setting incorrect surplus transfer percentages or auction bid sizes may lead to issues.

3. The CamelotRelayer.sol retrieves price information from an external source, the CamelotPair. Systemic risk exists if this source provides inaccurate or manipulated data, leading to incorrect price feeds and potentially affecting other parts of the protocol relying on this data and this contract converts and scales price results by manipulating decimals, if there are errors or discrepancies in this conversion, leading to incorrect pricing information and potentially impacting the entire system.

4. The "build" function allows users in Vault721 contract to create ODProxy contracts, but it can be abused. It does not have any checks or permissions other than verifying if a user already has an ODProxy. There is potential for abuse if users create excessive proxies, which could lead to misuse or spam.

5. In BasicActions.sol contract systemic risks might arise from the exit mechanisms, especially if there are discrepancies between the rates used in calculations and those from the external SAFE engine.

### Centralization Risks:

1. In AccountingEngine.sol contract allows specific accounts to be authorized and to modify parameters. Centralization risks are present if a small number of entities or individuals control these authorizations, potentially leading to centralized control over the protocol.

2. CamelotRelayer contract's constructor sets critical parameters like baseToken, quoteToken, baseAmount, multiplier, and quotePeriod. Centralization risks are present because these parameters are not governed by a decentralized process and are hardcoded and The `getResultWithValidity` function returns false if the pool doesn't have enough history. The validity of the result depends on the pool's history and may be influenced by external factors, introducing centralization risks.

3. The contract UniV3Relayer.sol doesn't provide a way to modify the fee tier or other pool-related parameters after deployment, which could be problematic if adjustments are needed based on changing market conditions.

4. The ODGovernor.sol specifies a fixed quorum fraction (3) that is not governed or adjustable through governance processes. Centralization risks arise as this parameter cannot be modified in response to changing conditions.

5. The owner's address is set as immutable in ODProxy.sol contract, meaning it cannot be changed once the contract is deployed. This lack of flexibility to change ownership through a decentralized process can be a centralization risk, as there is no governance mechanism in place to handle ownership transitions and the contract does not have a mechanism for upgrades or modifications. If there are bugs or vulnerabilities discovered in the contract's code, there is no way to implement improvements without deploying an entirely new contract, which can be cumbersome and disruptive.

**Properly managing these risks and implementing best practices in security and decentralization will contribute to the sustainability and long-term success of the Open Dollar protocol.**

### Conclusion

In general, the Open Dollar project exhibits an interesting and well-developed architecture we believe the team has done a good job
regarding the code, but the identified risks need to be addressed, and measures should be implemented to protect the protocol from
potential malicious use cases. Additionally, it is recommended to improve the documentation and comments in the code to enhance
understanding and collaboration among developers. It is also highly recommended that the team continues to invest in security
measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.

### Time spent:
14 hours
