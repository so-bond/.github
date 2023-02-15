# Sustainable & Open | Bond - open source contribution


From the [Compliant Architecture for Security Token  / CAST](https://www.cast-framework.com/wp-content/uploads/2021/05/CAST-White-Paper-1.0_Final_17-05-2021.pdf) proposed by Société Générale Forge in May 2021 and following a joint effort of Crédit Agricole CIB (CACIB) and Skandinaviska Enskilda Banken (SEB) to propose a modular bond issuance model on DLT, CACIB and SEB propose to extend the CAST framework with `so|bond` (Sustainable & Open model for bonds)

Rather than theorizing the model `so|bond` proposes an implementation of smart contracts under the Ethereum Virtual Machine (EVM) de-facto standard and release the source code of its implementation in [sc-bonds](https://github.com/so-bond/sc-bonds)

So, unlike CAST, `so|bond` is not DLT agnostic and instead takes a view of a ready operational standard for the market. However, since this position can prove to be invalid at some point, the `so|bond` model remain valid under similar blockchain technologies that have smart contracts with certain attributes.

There are commonalities and differences with the CAST Framework but `so|bond` has been inspired by CAST.

## Legal, operational and non technical preamble
`so|bond` does not impose a legal structure yet it assumes that the law accepts that the register of a security is kept in DLT. This documentation does not dive in this field.

`so|bond` however take the view that all operational actors must be independant and autonomous as much as possible to transact, decide their business workflows and custody their assets (and therefore their private keys). For this reason `so|bond` is prefering to isolate the roles in the bond issuance life cycle as much as possible (issuer, joint lead managers, billing and delivery agent, registrar - CAK in luxembourg law, custodians, investors, paying agent etc)

`so|bond` assumes that roles taken by the parties in a transaction, that need to operate directly in the DLT, understand that interacting with a smart contract implies that they understand the smart contract and accept its terms. In legal terms, although it has not the same value (yet), it would mean that at the first interacting with a smart contract (either by deployment or by submitting a transaction to it) the terms and conditions of the software (representing the smart contract) are deemed to be accepted. 


## Transfer management principle

Referring to CAST Framework chapter 1 section 2.5 Understanding transfer management principles, in `so|bond` we provide a peer to peer transfer solution that complies with regulation and enables the removal of a third party intermediary in the transaction.

This is possible by a technical approach that EVM support : smart contract code hash.

The Register, being the smart contract that holds the security's balances, needs to be operated via a call to a transfer function to perform the change of ownership. This function is configured to only accept calls from another smart contract the code of which is pre-approved by the Registrar (the party in charge of the Register). We call these approved smart contract "satellite".

By this mechanism, the Registrar can be sure that the calling satellite smart contract implements the appropriate checks before requesting the transfer of ownership. In particular the verification that both parties of the transfer agree to it.

Note that point 2.5 2) Possibility of adapting to technological or regulatory constraints remains applicable in `so|bond`.

## Roles adjustments compare to CAST

Referring to CAST Framework at chapter 2 section 1 we adjust the definitions to redistribute responsibilities of the Registrar further and clarify some points:

|Role | Responsibilities|
|-----|-----------------|
|**Issuer** | no change|
|**Structuring Manager** |no change. Can be embodied by several actors in case of a syndicated transaction. Can develop the Register smart contract or delegate it to a third party |
|**Lead Manager** |no change. Also called Billing & Delivery Agent in certain processes |
|**Registrar** | <li> Is not necessarilly developping the Security Tokens smart contract (or Register). However the Registrar must be in capacity to comprehend and validate the content of the code  |
|(or CAK)  | <li> Provides on-chain registration of the Security Tokens on the DLT before issuance and ensure execution transfer are correctly done (but not necessarilly doing it itself)  |
|          | <li> Acts as a securities account provider to the Security Token holders and as such perform duties required under the applicable legislation.  |
|          | <li> Grant roles in the register for other parties to perform services. A party is identified by the address of the party's private key |
|          | <li> Review and approve the form of satellite smart contract that it authorize to acts on the register on its behalf |
|          | <li> Does not necessarilly process notification, payments, coupon events etc as these can be delegated to other parties directly acting on the register|
|**Fiscal Agent**  |no change |
|**Calculation Agent** |no change |
|**Settlement Agent** | also Paying Agent. Acts for the cash settlement of coupon and redemption. But does not intervene in the payment against securities|
|**Custodian** |no change. they are typically the Security Token holders directly with the Registrar in a nominative administered bond.  |
|**Issuance Facility Operator** | not a formal role in the `so\|bond` model. Each role must be able to operate on the DLT directly or via a supplier but no central operator is expected.|
|**OTC Facility Operator** |not needed in `so\|bond` model. Custodian and Lead Manager are expected to deliver their order directly to the DLT in the appropriate form of satellite smart contract|

## Focus on transaction management for Security Tokens

Referring to the CAST Framework chapter 2 section 2.1 `so|bond` is aligned with the order, trade and legs principle as this section does not define how the execution of orders matching is done in practice. In `so|bond` also it remains free but it is designed to be implemented in satellite  smart contracts and not in the Register.

## Service map

Compared to the CAST Framework (chapter 2 section 3) , `so|bond` does not cover the trading part as it is consider that on-chain trading would not be relevant in the medium term. The Pilot Regime will allow to furter explore this, but matching order books in a DLT is knwon to be opening the possibility of front running by node operators.

All the process flow described in this section are sound but `so|bond` model propose to be more open and flexible in these processes and to let organisation emerge and innovate rather than limiting to a set of defined models.   
The `so|bond` model expects that workflows to be implemented are agreed between the parties and encoded into satellite smart contracts. So the CAST Framework is an appropriate base for such processes.


## Technical framework

Following chapter 4 of the CAST Framework

### Instrument registry pre-requisites

in `so|bond` the Instrument registry is not a requirement and can be replaced with benefits by events generated by the individual instrument smart contract that can be collected by any actor easilly. However if used as a complement it can follow the CAST approach.

### Security Token smart contract (or Register)

in `so|bond` the Security Tokens’ Smart Contract (or Register) must have the following modules in its code

* Instrument Term Sheet: defines like in CAST all financial attributes, and future event dates of the bond but not the coupon calculation logic or future cash flows.
* Operator management: similar to cast with the use of role. `so|bond` recommend the use of the `openzeppelin/access library`.
* Balance management: same as CAST. With in addition the capacity for the allowed satellite contracts to request the creating of balance snapshot using the `openzeppelin/contracts/token/ERC20/extensions/ERC20Snapshot` library. Snapshot is an important feature to enable keeping the record of the owners at a certain point in time
* Satellite management: define methods to allow or remove satellite smart contracts

As `so|bond` focuses on implementation, it also recommand the following types for the various data
* date and time are expressed in seconds since 1/1/1970 in GMT base. This is important because the nodes of the DLT can run in any time zone and they should share an absolute reference of time. UI are capable of converting easilly such time and date in local form
* financial amount are expressed in integer form, with as many decimal significance as the currency requires. For instance 123,45€ will be stored as 12345.
* interest rate are expressed in integer form multiplied by 10 000. 1% will therefore be represented as 10000 therefore with 4 available decimals or 0,01 basis point.

All operations on numeric data are done with integers, prioritizing the multiplication over the division (to avoid loosing precision). The values are manipulated in their integer form. Division is done in the Euclidian form and that implies that *half a cent is rounded to zero* (to keep in mind in legal documentation)

Below the proposed interface of the 4 modules

#### Instrument Term Sheet

```javascript
interface IRegisterTS {
  enum Status {Draft, Ready, Issued, Repaid, Frozen}
  function status() external view returns (Status);
  // The event is used to quickly find existing bonds and their status history. It replaced the Instrument Registry
  event RegisterStatusChanged( string indexed isin, Status indexed status, string name );
  // swich status from Issued to Frozen and vice versa
  function toggleFrozen() external;

  struct BondData {
      string name;
      string isin; // initialized by the contructor. Also set in the ERC20 symbol
      uint256 expectedSupply; // is the number of unit of bond - the minted amount when Registrar calls makeReady()
      bytes3 currency; //currency as 3 characters ISO code 
      uint256 unitValue; //value of 1 unit of bond in currency
      uint256 couponRate; //percentage of interests per year (eg 0.25%).
      uint256 creationDate; //Actual date of creation 
      uint256 issuanceDate; //The actual date of issuance (when the bond start to exists legally speaking)
      uint256 maturityDate; //The expected date in the future for the repayment by the issuer - expected to be issuance date + N years
      uint256[] couponDates; //An array of dates, as anniversary dates of the issuance until maturity (excluded). 
      uint256 cutOffTime; //the time part of the coupon snapshot (must be lower than 24*3600 sec)
  }
  function setIsinSymbol(string memory isinSymbol) external;
  function getBondData() external view returns (BondData memory);
  function setBondData(
      string memory name,
      uint256 expectedSupply,
      bytes3 currency,
      uint256 unitVal,
      uint256 couponRate,
      uint256 issuanceDate,
      uint256 maturityDate,
      uint256 cutOffTime
    ) external;
  function addCouponDate(uint256 date) external;
  function delCouponDate(uint256 date) external;

}
```

#### Operator management

```javascript
import "@openzeppelin/contracts/access/AccessControl.sol";
interface IRegisterRoleManagement is AccessControl {
  // control the master admin role. requires 2 different registrar wallet to effect the change
  function changeAdminRole(address account) external; 
}
```

#### Balance management


```javascript
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Snapshot.sol";
interface IRegisterBalanceManagement is ERC20Snapshot {
  // allow and remove investor wallet in/from the list of allowed wallets (form of KYC status and sancionned party)
  function enableInvestorToWhitelist(address investor);
  function disableInvestorFromWhitelist(address investor); 
  function investorsAllowed(address investor) view returns (bool);
  function getAllInvestors() view returns (address[] memory);

  // Defines when the next snapshot should be taken. Called by satellite smart contract
  // This sets a timestamp at couponDate + cutOffTime that get tested during a transfer to decide if a snapshot should be taken before updating the balance
  function setCurrentCouponDate(uint256 couponDate, uint256 cutOffTime);

  // current balance of the given investor's account
  function balanceOf(address investor);
  // current balance of the given investor's account at the specific coupon date in the past
  function balanceOfCoupon(address investor, uint256 couponDate);

  // mint the expected supply to the technical wallet and change state from Draft to Ready (for issuance)
  function makeReady();
  // execute a transfer between two accounts, only for the registrar or via satellite smart contracts
  function transferFrom( address from, address to, uint256 amount) returns (bool);
  // execute a transfer of the total balance of the investor to the technical wallet to perform the redemption to be called by satellite smart contracts
  function returnBalanceToPrimaryIssuanceAccount(address investor) returns (bool);
}
```

#### Satellite management

These functions are limited to the use of the Registrar.

```javascript
interface IRegisterSatelliteManagement {

  // allow any smart contract with the provided runtime code hash. Attention code hash does not cover the smart contract constructor
  function enableContractToWhitelist(bytes32 contractHash);
  // disable a previously enabled runtime code hash
  function disableContractFromWhitelist(bytes32 contractHash);
  // return true if the caller is a smart contract that was whitelisted
  function isCallerApprovedSmartContract() view returns(bool);
}
```

**Important note:** The Registrar, being responsible for the maintenance of the records of securities tokens and ensuring that transfers are properly done, that corporate actions are possible and that there is no loss or duplication of securities during its lifetime, must control what satellite smart contracts can do.    
When a third party creates a satellite smart contract and wishes it to be approved by the Registrar it will provide the Registrar 
- a functional and detailled explanation of the need and proposed implementation
- the solidity code of the smart contract and the compilator version (+ optimisation options)
- an eternal audit of the smart contract (or a formal proof of the code)
- the runtime code hash of the smart contract

The Registrar will have to verify that
- the functionality proposed does not create a mis use or endanger the Register
- that all parties involved in the satellite smart contracts at runtime have agreed to the terms of it before interacting with the Register
- that the compilation and the hash of the runtime code does match the provided hash.

### Trade smart contract - generic interface

In `so|bond` model the belief is that business processes will evolve rather quickly (new ideas, different cash on chain solutions, CBDC ...) and therefore transaction workflow implemented in smart contract should be replaceable. As a consequence, the `so|bond` model prefers single smart contract instance for each transaction with a short lived process. With this approach the next trade can have a different process while still interfacing the same register. It is however possible that the transactions are managed by a factory model or even by a facade smart contract.

There is the need of being able to retrieve the trades and to get their details in an homogeneous way. To do this `so|bond` proposes a common interface for trades.

```javascript
interface ITrade {
  // states of a trade. Pending means approved by one party, Accepted means approved by the other party and Executed means the security is transfered
  enum Status {Draft, Pending, Rejected, Accepted, Executed, Paid}
  
  struct TradeDetail {
    uint256 quantity;
    address buyer;
    uint256 tradeDate;
    uint256 valueDate;
    uint256 price;
  }
  // on what register this trade acts into
  function register() external view returns (IRegister);
  function status() external view returns (Status);
  function paymentID() external view returns (bytes8);
  function getDetails() external view returns(TradeDetail memory);
  function sellerAccount() external view returns (address);
  function buyerAccount() external view returns (address);

  // this event is used to retrieve trades from the DLT log
  event NotifyTrade(address indexed seller, address indexed buyer, Status indexed status, uint256 quantity);
}
```

### Primary Issuance Satellite Smart contract
As an initial proposal in the `so|bond` model for the process of issuance, a.k.a. the first actual purchase of the securities from the issuer to the Lead Manager is executed by a PrimaryIssuance smart contract that is deployed by the Lead Manager, pointing to the register.    
The PrimaryIssuance smart contract will follow the ITrade interface but enfore to only execute a transfer the total quantity of securities from the technical initial wallet and place it in the Lead Manager Wallet. The Registrar will verify that the smart contract verifies that it is the case and that the call is made only by the Lead Manager's wallet.    

### Bilateral Trade Satellite Smart contract
When two investors want to exchange securities they setup a new bilateral transaction represented by a smart contract that include the agreed process.

For instance, taking the convention that it is always the seller that initializes the smart contract, and approves it. The buyer get notified by the `NotifyTrade` event, verifies and confirm the trade, locking the properties. The payment can be done outside of the trade and when the payment is received the seller finalizes the execution by triggering via the contract the `transferFrom()` function on the register. The Registrar will check that the code of the contract controls that both parties have to confirm the trade, and that the transfer can only be done on properties agreed by both parties.    
Obviously in this simplistic process there is a delivery risk that comes from the lack of on-chain cash.

In a more sofisticated process, the smart contract could contains reference to on-chain settlement assets (stable coins or on-chain cash accounts) without having to change anything to the register.

## Business Continuity

## Oracle or integration layer

