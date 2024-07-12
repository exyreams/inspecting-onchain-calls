## KyberSwap - UpgradeabilityProxy.sol

### Introduction

**Protocol Name:** KyberSwap  
**Category:** DeFi  
**Smart Contract:** UpgradeabilityProxy.sol

### Function Analysis
**Function :** Constructor  
**Block Explorer Link:**  [https://etherscan.io/token/0xdeFA4e8a7bcBA345F687a2f1456F5Edd9CE97202#code](https://etherscan.io/token/0xdeFA4e8a7bcBA345F687a2f1456F5Edd9CE97202#code)  
**Function Code:**

`constructor(address _logic, bytes memory _data) public payable {
    assert(IMPLEMENTATION_SLOT == bytes32(uint256(keccak256("eip1967.proxy.implementation")) - 1));
    _setImplementation(_logic);
    if (_data.length > 0) {
        (bool success,) = _logic.delegatecall(_data);
        require(success);
    } }`

**Used Encoding/Decoding or Call Method:** `delegatecall`

### Explanation

**Purpose:**  
`constructor` function set implementation & proxied contract initially by calling function on implementation contract with given data.

**Detailed Usage:**  

`delegatecall` is used to call function implementation contract by using context of proxy contract &  `_logic.delegatecall(_data)` is called if `_data` is provided. it allows proxy to run initilization function on implementation contract while passing necessary parameters encoded in `_data`. 

Use of `delegatecall` ensure storage and context to be within proxy contract while executing logic defined in implementation contract which enables proxy contract to be upgradable as it can be changed without affecting proxy's state.

**Impact:**  

`delegatecall` is important for initializing proxy contract with specific logic from implementation cotnract. This allows proxy to start with predfined configuration & state making it versatile for using immediately after deployment. it also allows upgradabeability feature, allowing proxy to delegate calls from various implementations over time while maintaining state and address.

**UpgradeabilityProxy.sol Contact Source Code:**

    // SPDX-License-Identifier: MIT
    
    pragma solidity ^0.6.0;
    
    import './Proxy.sol';
    import '@openzeppelin/contracts/utils/Address.sol';
    
    /**
     * @title UpgradeabilityProxy
     * @dev This contract implements a proxy that allows to change the
     * implementation address to which it will delegate.
     * Such a change is called an implementation upgrade.
     */
    contract UpgradeabilityProxy is Proxy {
      /**
       * @dev Contract constructor.
       * @param _logic Address of the initial implementation.
       * @param _data Data to send as msg.data to the implementation to initialize the proxied contract.
       * It should include the signature and the parameters of the function to be called, as described in
       * https://solidity.readthedocs.io/en/v0.4.24/abi-spec.html#function-selector-and-argument-encoding.
       * This parameter is optional, if no data is given the initialization call to proxied contract will be skipped.
       */
      constructor(address _logic, bytes memory _data) public payable {
        assert(IMPLEMENTATION_SLOT == bytes32(uint256(keccak256('eip1967.proxy.implementation')) - 1));
        _setImplementation(_logic);
        if(_data.length > 0) {
          (bool success,) = _logic.delegatecall(_data);
          require(success);
        }
      }  
    
      /**
       * @dev Emitted when the implementation is upgraded.
       * @param implementation Address of the new implementation.
       */
      event Upgraded(address indexed implementation);
    
      /**
       * @dev Storage slot with the address of the current implementation.
       * This is the keccak-256 hash of "eip1967.proxy.implementation" subtracted by 1, and is
       * validated in the constructor.
       */
      bytes32 internal constant IMPLEMENTATION_SLOT = 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc;
    
      /**
       * @dev Returns the current implementation.
       * @return impl Address of the current implementation
       */
      function _implementation() internal override view returns (address impl) {
        bytes32 slot = IMPLEMENTATION_SLOT;
        assembly {
          impl := sload(slot)
        }
      }
    
      /**
       * @dev Upgrades the proxy to a new implementation.
       * @param newImplementation Address of the new implementation.
       */
      function _upgradeTo(address newImplementation) internal {
        _setImplementation(newImplementation);
        emit Upgraded(newImplementation);
      }
    
      /**
       * @dev Sets the implementation address of the proxy.
       * @param newImplementation Address of the new implementation.
       */
      function _setImplementation(address newImplementation) internal {
        require(Address.isContract(newImplementation), "Cannot set a proxy implementation to a non-contract address");
    
        bytes32 slot = IMPLEMENTATION_SLOT;
    
        assembly {
          sstore(slot, newImplementation)
        }
      }
    }
