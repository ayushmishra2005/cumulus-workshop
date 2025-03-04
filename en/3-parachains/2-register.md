# Parachain Registration

We have our relay chain launched and our parachain collator ready to go. Now we have to register the
parachain on the relay chain. In the live Polkadot network, this will be accomplished with
[parachain auctions](https://wiki.polkadot.network/docs/en/learn-auction). But today we will do it
with Sudo.

## Registration Transaction

The transaction can be submitted **on a relay chain node** from 
`Developer > Sudo > parasSudoWrapper > sudoScheduleParaInitialize` with the following parameters:

- id: `200`
- genesisHead: upload the file `para-200-genesis` (from the previous step)
- validationCode: upload the file `para-200-wasm` (from the previous step)
- parachain: Yes

![Registration screenshot](../../assets/img/registration-screenshot.png)

This successful dispatch will emit the `sudo.Sudid` event, viewable in the relay chain explorer page.

If you are running a network with more than two validators you can add more parachains through the
same interface with the parameters adjusted accordingly. More important details on this in the
[latter on in this tutorial](en/3-parachains/4-more-nodes).



### Block Production

The collator should start producing parachain blocks (aka collating) once the registration is
successful. **Look in the block explorer tab on the Apps ui for a `parasInherent.enter` event -**
**this signals that the parachain is now active**.

> This may take a while! be patient as you need to wait for a new session to begin first.
> This is 10 blocks for the 
> [included rococo `chain-spec.json`](en/1-prep/2-chain-spec?id=_1a-using-a-prebuilt-chain-spec) 
> in this workshop's files.

Finally, the collator should start producing log messages like the following:

```
2021-01-14 16:09:54  [Relaychain] ✨ Imported #519 (0x7c22…71b8)
2021-01-14 16:09:54  [Relaychain] Starting collation for relay parent 0x7c22474df9f10b44aed7616c3ad9aef4d0db82e8421a81cbc3c10e63569971b8 on parent 0x4d77beb48b42979b070e0e81357f66629da194faa0f72be0bb70ee6828c220d0.
2021-01-14 16:09:54  [Relaychain] 🙌 Starting consensus session on top of parent 0x4d77beb48b42979b070e0e81357f66629da194faa0f72be0bb70ee6828c220d0
2021-01-14 16:09:54  [Relaychain] 🎁 Prepared block for proposing at 18 [hash: 0x8cb3aa750b83e1dfc120c81243e8d7fdb3f6926adfe79b977ec7d8f4a5f7bb7b; parent_hash: 0x4d77…20d0; extrinsics (3): [0x9d73…3794, 0xd860…3108, 0x6fdb…0112]]
2021-01-14 16:09:54  [Relaychain] Produced proof-of-validity candidate 0x67b91f2a3e0cc82d0b18a2ec31212081853b24e5c8f7de98d39fabfd89f46bee from block 0x8cb3aa750b83e1dfc120c81243e8d7fdb3f6926adfe79b977ec7d8f4a5f7bb7b.
2021-01-14 16:09:54  [Parachain] ✨ Imported #18 (0x8cb3…bb7b)
2021-01-14 16:09:54  [Relaychain] 💤 Idle (4 peers), best: #519 (0x7c22…71b8), finalized #516 (0x982f…d9cf), ⬇ 239.4kiB/s ⬆ 239.6kiB/s
2021-01-14 16:09:55  [Parachain] 💤 Idle (0 peers), best: #17 (0x4d77…20d0), finalized #15 (0x25ec…a10b), ⬇ 633.5kiB/s ⬆ 622.3kiB/s
2021-01-14 16:09:59  [Relaychain] 💤 Idle (4 peers), best: #519 (0x7c22…71b8), finalized #517 (0x6852…ec17), ⬇ 216.3kiB/s ⬆ 216.6kiB/s
2021-01-14 16:10:00  [Relaychain] ✨ Imported #520 (0x0ecb…4dba)
2021-01-14 16:10:00  [Parachain] 💤 Idle (0 peers), best: #17 (0x4d77…20d0), finalized #16 (0xd7e0…ae67), ⬇ 503.7kiB/s ⬆ 494.3kiB/s
2021-01-14 16:10:04  [Relaychain] 💤 Idle (4 peers), best: #520 (0x0ecb…4dba), finalized #518 (0x15df…f3fa), ⬇ 282.0kiB/s ⬆ 275.3kiB/s
2021-01-14 16:10:05  [Parachain] 💤 Idle (0 peers), best: #17 (0x4d77…20d0), finalized #16 (0xd7e0…ae67), ⬇ 605.2kiB/s ⬆ 595.0kiB/s
```

#### Collator Data Base Corruption or Loss

> NOTE: your sole collator is the _only home of all parachain data_ as there is only one node
> on your entire network! Relay chains only store _header_ information! If the parachian DB is lost
> (my using `--tmp` for you collator as an example) you will **NOT** be able to recover the chain!

If you _must_ purge your chain, you need to deregister and re-register! It may be easier in testing
to instead purge all the chains. To purge the collator DB run:

```bash
# Purge the collator(s)
parachain-collator purge-chain\
  --base-path <your collator DB path set above> \  # <-- set a proper path

# Purge the validator(s)
polkadot purge-chain\
  --base-path <your collator DB path set above> \  # <-- set a proper path
```

Then register from a [blank slate](#registration-transaction) again.

### Parachian Block Finalization

The relay chain tracks the latest blocks (the heads) of each parachain. When a relay chain block
is finalized, any parachain blocks that have completed the
[validation process](https://polkadot.network/the-path-of-a-parachain-block/) are also finalized.
This is how Polkadot achieves shared security.

We can keep track of what parachains are registered and what their latest head data is on the
`Network > Parachains` tab in the Apps UI.

![Parachain Head Information](../../assets/img/parachain-summary-screenshot.png)