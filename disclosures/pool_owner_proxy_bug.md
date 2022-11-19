# Vulnerability disclosure 2022-11-20

## Summary

- A vulnerability was discovered in an old [Curve Pool Owner contract](https://etherscan.io/address/0x8cf8af108b3b46ddc6ad596aebb917e053f0d72b). This contract was the admin of the old stableswap factory ([0x0959158b6040D32d04c301A72CBFD6b39E21c9AE](https://etherscan.io/address/0x0959158b6040D32d04c301A72CBFD6b39E21c9AE)).
- The vulnerability did not affect user funds. The attack vector was limited to fees earned by the DAO from a minimal set of pools (metapools created by the old factory). 
- The bug reporter was paid out a bounty of 100k CRV.
- Total value lost: approximately 18k USD of DAO fees. This loss could have been avoided entirely but was exploited by the individual who reported the bug. They have been reported to ImmuneFi* for failing to return these funds.
- The vulnerability has been patched via a DAO parameter vote: [Curve Parameter vote 46](https://dao.curve.fi/vote/parameter/46).
- All funds are safe.

*At the time of writing this report, Curve does not have a bug bounty under ImmuneFi. The security researcher was investigating another protocol (Beanstalk) with a liquidity pool on Curve, where they discovered this bug.

## Background

On 21st October 2022, an independent security researcher discovered an unguarded contract method in a Pool Owner contract that allowed anyone to set the `fee_receiver` of metapools in the old stableswap factory. Following this report, Curve core contributors created a new Pool Owner contract to fix issues presented in the faulty Pool Owner contract. Including it into the Curve framework required a DAO vote. The vulnerability has since been patched.

Before and after reporting the vulnerability, the security researcher actively exploited the bug and profited approximately $18k (from DAO fees). They have since refused to return these funds, despite receiving a generous bounty. As the researcher works under the ImmuneFi umbrella, Curve core contributors have reported this behaviour to the ImmuneFi team.


## Details of the vulnerability

The faulty contract was: [Old Pool Owner](https://etherscan.deth.net/address/0x8cf8af108b3b46ddc6ad596aebb917e053f0d72b). The Pool Owner contract is responsible for setting and changing pool parameters (which do not affect user funds) and receiving fees accrued from swaps (which also do not affect user funds). The pool owner was the admin of the [Old Stableswap Factory](https://etherscan.io/address/0x0959158b6040D32d04c301A72CBFD6b39E21c9AE).

The bug was an unguarded method:

```
@external
def set_fee_receiver(_target: address, _base_pool: address, _fee_receiver: address):
    Factory(_target).set_fee_receiver(_base_pool, _fee_receiver)
```

This bug allowed anyone to set a specific address (`_fee_receiver` parameter) as the fee recipient of pools paired with `_base_pool` (e.g. 3pool). The bug only affected pools deployed by the old factory. The old factory was already deprecated in favour of the [New Stableswap Factory](https://etherscan.io/address/0xB9fC157394Af804a3578134A6585C0dc9cc990d4) a while ago. In this case, the target was the Old Stableswap factory: (tx hash)[https://etherscan.io/tx/0xe1fc066f789bcf17e22448d814c8f909a788f186a3d875062edb434857d3c35e].

### Exploit contract

This bug did not require an external contract to exploit the bug.

## Details of the fix

The fix involved two steps:

1. Replace the admin of the Old Stableswap Factory with a new contract that fixes this vulnerability: [0x8c392fB6F79e2564D73Fe13FB3Ef034f5A309c3D](https://etherscan.io/address/0x8c392fB6F79e2564D73Fe13FB3Ef034f5A309c3D).
2. Set fee recipient to [0xeCb456EA5365865EbAb8a2661B0c503410e9B347](https://etherscan.io/address/0xeCb456EA5365865EbAb8a2661B0c503410e9B347).

The Parameter vote that fixes these issues: [Parameter VoteID: 46](https://etherscan.io/tx/0x97d54c529af1c4ea989c08647b6a9b66bc430e77c94516238a98289b83e899a8)

The enactment: [0x98a86c5b09ee1e752a9d74ce08c1fe74229e13cd9a9f871a90cda67ad4a16fe3](https://etherscan.io/tx/0x98a86c5b09ee1e752a9d74ce08c1fe74229e13cd9a9f871a90cda67ad4a16fe3).

## Timeline

- While investigating bugs for the Beanstalk protocol (which funds an ImmuneFi program), the security researcher finds the unguarded `set_fee_receiver` method in the old Pool owner contract.
- They exploit it and set their address ([0xd2bafa7f3ac35b38ca4a67e07cbbab1ac0f3796b](https://etherscan.io/address/0xd2bafa7f3ac35b38ca4a67e07cbbab1ac0f3796b)) as the new fee receiver: [0xe1fc066f789bcf17e22448d814c8f909a788f186a3d875062edb434857d3c35e](https://etherscan.io/tx/0xe1fc066f789bcf17e22448d814c8f909a788f186a3d875062edb434857d3c35e).
- The security researcher then extracts DAO fees from several pools:
  - [FRAX <> 3CRV](https://etherscan.io/tx/0x1c0ce747fbb0c0d65499b42b57d1857291e6e7f36c2e6e5fff949eb20783af1b)
  - [alUSD <> 3CRV](https://etherscan.io/tx/0x5993fa9c141545e31f76385304c66d573526e91c9560722ff34c4183e1ede601)
  - [BEAN <> 3CRV](https://etherscan.io/tx/0x920134ffe50ab4534f591cef44ec203474b5d7dd35ad9f8e50d2a13fa1c4bf4a)
- They then sell the exploited tokens on Uniswap:
  - [0xab3f08620bb3f7d388e52ca68e4b38778847a99b09957fde6670e5dd5985c282](https://etherscan.io/tx/0xab3f08620bb3f7d388e52ca68e4b38778847a99b09957fde6670e5dd5985c282)
  - [0x0d93ac6943b6cb3f40b67dd4799911f4d54dd9e95a6273fe5fa03deec5f5bc1f](https://etherscan.io/tx/0x0d93ac6943b6cb3f40b67dd4799911f4d54dd9e95a6273fe5fa03deec5f5bc1f)
  - [0xd3397312a852c7ff39af15d1cf267ede0b75a19450a292d245e00be8a591abff](https://etherscan.io/tx/0xd3397312a852c7ff39af15d1cf267ede0b75a19450a292d245e00be8a591abff)
- Having exploited fees from the Curve DAO, they raise the vulnerability to security@curve.fi and on the #dev channel on Curve Finance discord.
- The individual is paid out 100k CRV for reporting the bug: (0xc78ba0c03e6aca0e12115019bd8e63792660276ad460a0921b0b971352593349)[https://etherscan.io/tx/0xc78ba0c03e6aca0e12115019bd8e63792660276ad460a0921b0b971352593349]
- Following discussions with the team, an amended contract is created, and a DAO vote is initiated and enacted.
- During the voting period, the researcher's EOA receives claimed fees from the bot that claims and burns fees for Curve. We ask the individual to return the funds, but they refuse.
- Classifying this as black hat behaviour, the individual is reported to ImmuneFi.


## Links

The blackhat's addresses:
- [0xd2bafa7f3ac35b38ca4a67e07cbbab1ac0f3796b](https://etherscan.io/address/0xd2bafa7f3ac35b38ca4a67e07cbbab1ac0f3796b)
- [0xf6403426e73534c8fbce1c0d39ceda126d1cffa7](https://etherscan.io/address/0xf6403426e73534c8fbce1c0d39ceda126d1cffa7)

The blackhat has transferred funds that require KYC:

- [Coinbase](https://etherscan.io/tx/0xed69790397cf759c0fd7f2a3b9476e41a39ce55f094266d243b9ea6ed624781e)
- [HitBTC](https://etherscan.io/tx/0x895be67cf719762c0aac4fb588a934f91c5015185e0cfe7225374a23285b3f5c)
- [Binance](https://etherscan.io/tx/0x30a0bc9f253151a196aee544d7a37d7e6bc2e632d919c6b7c92313c143bd34f8)
- [FTX](https://etherscan.io/tx/0xc8094c35aa619ebd9998c2ca104cc1f176406c7a032400d1a4fc7c2271a8df7f)
