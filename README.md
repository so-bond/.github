# Sustainable Open | Bond - open source contribution


From the [Compliant Architecture for Security Tken  / CAST](https://www.cast-framework.com/wp-content/uploads/2021/05/CAST-White-Paper-1.0_Final_17-05-2021.pdf) proposed by Société Générale Forge in May 2021 and following a joint effort of Crédit Agricole CIB (CACIB) and Skandinaviska Enskilda Banken (SEB) to propose a modular bond issuance model on DLT, CACIB and SEB propose to extend the CAST framework with `so|bond` (Sustainable Open model for bonds)

Rather than theorizing the model `so|bond` proposes an implementation of smart contracts under the Ethereum Virtual Machine (EVM) de-facto standard and release the source code of its implementation in [sc-bonds](../sc-bonds/README.md)

So, unlike CAST, `so|bond` is not DLT agnostic and instead takes a view of a ready operational standard for the market. However, since this position can prove to be invalid at some point, the `so|bond` model remain valid under similar blockchain technologies that have smart contracts with certain attributes.

There are commonalities and differences with the CAST Framework but `so|bond` has been inspired by CAST.

## Legal, operational and non technical preamble
`so|bond` does not impose a legal structure yet it assumes that the law accepts that the register of a security is kept in DLT. This documentation does not dive in this field.

`so|bond` however take the view that all operational actors must be independant and autonomous as much as possible to transact, decide their business workflows and custody their assets (and therefore their private keys). For this reason `so|bond` is prefering to isolate the roles in the bond issuance life cycle as much as possible (issuer, joint lead managers, billing and delivery agent, registrar - CAK in luxembourg law, custodians, investors, paying agent etc)

`so|bond` assumes that roles taken by the parties in a transaction, that need to operate directly in the DLT, understand that interacting with a smart contract implies that they understand the smart contract and accept its terms. In legal terms, although it has not the same value (yet), it would mean that at the first interacting with a smart contract (either by deployment or by submitting a transaction to it) the terms and conditions of the software (representing the smart contract) are deemed to be accepted. 

## Technical components

### Transfer management principle

Referring to CAST Framework at chapter 2.5 Understanding transfer management principles, in `so|bond` we provide a peer to peer transfer solution that complies with regulation and enables the removal of a third party intermediary in the transaction.

This is possible by a technical approach that EVM support : smart contract code hash.

The Register, being the smart contract that holds the security's balances, needs to be operated via a call to a transfer function to perform the change of ownership. This function is configured to only accept calls from another smart contract the code of which is pre-approved by the Registrar (the party in charge of the Register).

By this mechanism, the Registrar can be sure that the calling smart contract implements the appropriate checks before requesting the transfer of ownership. In particular the verification that both parties of the transfer agree to it.

Note that point 2.5 2) Possibility of adapting to technological or regulatory constraints remains applicable in `so|bond`.

