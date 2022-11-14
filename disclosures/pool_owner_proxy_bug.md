# Vulnerability disclosure 2022-11-20

## Summary

- A vulnerability was discovered in an old [Curve Pool Owner contract](https://etherscan.io/address/0x8cf8af108b3b46ddc6ad596aebb917e053f0d72b).
- The vulnerability did not affect user funds, and only affected fees earned by the DAO by a very small set of pools.
- Total value lost: approximately 18k USD: this was paid out to the security researcher who found and reported the vulnerability.
- The vulnerability has been patched via a DAO parameter vote: [Curve Parameter vote 46](https://dao.curve.fi/vote/parameter/46).
- All funds are safe. No action is required by the user.

## Background

On 21st October, 2022, a security researcher under the ImmuneFi umbrella discovered an unguarded contract method that allowed to set `fee_receiver` of pools paired with 3CRV in the old factory: [0x0959158b6040D32d04c301A72CBFD6b39E21c9AE](https://etherscan.io/address/0x0959158b6040D32d04c301A72CBFD6b39E21c9AE).

Following the discovery of this vulnerability, the researcher then tested it out on mainnet (we do not recommend this), thereby setting their EOA as the fee receiver of pools such as alUSD <> 3CRV, FRAX <> 3CRV etc. Following this, they claimed admin fees for several pools to test that the vulnerability exists and even proceeded to swap the accrued fees (in 3CRV) to USDC (we do not recommend this).

Having set themselves as the fee receiver, and having claimed fees from two pools (<1100 3CRV),

1. [FRAX <> 3CRV](https://etherscan.io/tx/0x1c0ce747fbb0c0d65499b42b57d1857291e6e7f36c2e6e5fff949eb20783af1b)
2. [alUSD <> 3CRV](https://etherscan.io/tx/0x5993fa9c141545e31f76385304c66d573526e91c9560722ff34c4183e1ede601)
3. [BEAN <> 3CRV](https://etherscan.io/tx/0x920134ffe50ab4534f591cef44ec203474b5d7dd35ad9f8e50d2a13fa1c4bf4a)

they then came to the Curve Finance discord and disclosed the vulnerability to one of the core developers. Following this disclosure, the core team proceeded to fix the vulnerability. Since the core team did not possess admin controls to replace the faulty Pool Owner contract with the correct one, a [DAO vote was proposed](https://dao.curve.fi/vote/parameter/46).

- All funds are safe. No action is required by the user.. Security disclosures could not be made since the vulnerability was exploitable by anyone until the admin of the old factory (0x0959158b6040D32d04c301A72CBFD6b39E21c9AE) was replaced.

The vulnerability has been since patched and the new owner of the old factory is the updated PoolOwner contract: [0x8c392fB6F79e2564D73Fe13FB3Ef034f5A309c3D](https://etherscan.io/address/0x8c392fB6F79e2564D73Fe13FB3Ef034f5A309c3D#code).

During the time that the vulnerability was open (between the submitting of the parameter vote to its enaction) the security researcher was the fee recepient from several pools. During a routine claim of admin fees from these small pools, the researcher's EOA received additional [16.37k 3CRV](https://etherscan.io/tx/0xd3397312a852c7ff39af15d1cf267ede0b75a19450a292d245e00be8a591abff) (post vulnerability disclosure), which they decided to keep despite having been paid a generous bounty of 100k CRV. Upon asking the researcher to pay it back, they refused and mentioned that they would return it if their trades went up in value.

We classified this behavior as blackhat, and have reported this behavior to the ImmuneFi team. They will now handle this internally. This has brought up the total DAO fees lost to the security researcher to 18k USD. We assume this to be lost, until ImmuneFi convince the researcher to return funds to Curve DAO.

This vulnerability does not affect user funds in pools in any way.

## Details of vulnerability

The faulty contract was: [Old Pool Owner](https://etherscan.deth.net/address/0x8cf8af108b3b46ddc6ad596aebb917e053f0d72b). The Pool Owner contract is responsible for setting and changing pool parameters (which do not affect user funds) and receiving fees accrued from swaps (which also do not affect user funds). The pool owner was the admin of the [Old Stableswap Factory](https://etherscan.io/address/0x0959158b6040D32d04c301A72CBFD6b39E21c9AE).

The bug was an unguarded method:

```
@external
def set_fee_receiver(_target: address, _base_pool: address, _fee_receiver: address):
    Factory(_target).set_fee_receiver(_base_pool, _fee_receiver)
```

which allowed anyone to set a specific address (`_fee_receiver` parameter) as the fee receipient of pools paired with `_base_pool` (e.g. 3pool). This ONLY affected pools deployed by the old factory. The old factory has already been deprecated in favor of the [New Stableswap Factory](https://etherscan.io/address/0xB9fC157394Af804a3578134A6585C0dc9cc990d4) a while ago. In this case, the target was the Old Stableswap factory: (tx hash)[https://etherscan.io/tx/0xe1fc066f789bcf17e22448d814c8f909a788f186a3d875062edb434857d3c35e].

## Details of fix

The fix involved two steps:

1. Replace the admin of the Old Stableswap Factory to a new contract that fixes this vulnerability: [0x8c392fB6F79e2564D73Fe13FB3Ef034f5A309c3D](https://etherscan.io/address/0x8c392fB6F79e2564D73Fe13FB3Ef034f5A309c3D).
2. Set fee receipient to [0xeCb456EA5365865EbAb8a2661B0c503410e9B347](https://etherscan.io/address/0xeCb456EA5365865EbAb8a2661B0c503410e9B347).

The Parameter vote that fixes these issues: [Parameter VoteID: 46](https://etherscan.io/tx/0x97d54c529af1c4ea989c08647b6a9b66bc430e77c94516238a98289b83e899a8)

The enactment: [0x98a86c5b09ee1e752a9d74ce08c1fe74229e13cd9a9f871a90cda67ad4a16fe3](https://etherscan.io/tx/0x98a86c5b09ee1e752a9d74ce08c1fe74229e13cd9a9f871a90cda67ad4a16fe3).

## Further actions

No further actions are needed.
