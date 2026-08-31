# DecentraVote — Interactive Demo Dashboard

**A zero-setup, browser-based demonstration of the DecentraVote blockchain-voting prototype.**

`Prototype` · `Simulated chain` · `No dependencies` · `Runs fully offline`

> DecentraVote is a national-hackathon prototype exploring how a smart contract and a public, verifiable ledger can move an election's trust away from a single central database and toward rules and records that participants can check for themselves. This dashboard lets anyone *see the idea run* — verify eligibility, cast a vote, watch it become a tamper-evident on-chain record, and independently audit the result — without installing anything or connecting a wallet.

---

## Contents

- [What it is](#what-it-is)
- [Use case](#use-case)
- [What it demonstrates](#what-it-demonstrates)
- [Quick start](#quick-start)
- [Guided demo walkthrough](#guided-demo-walkthrough)
- [The demo election](#the-demo-election)
- [How it works](#how-it-works)
- [The smart contract](#the-smart-contract)
- [Production stack vs this demo](#production-stack-vs-this-demo)
- [Honesty and limitations](#honesty-and-limitations)
- [Testing and QA](#testing-and-qa)
- [Files in this folder](#files-in-this-folder)
- [From prototype to production](#from-prototype-to-production)
- [Links](#links)
- [License](#license)

---

## What it is

`DecentraVote-demo.html` is a single, self-contained web page. Open it in any modern browser and it runs immediately — there is no build step, no server, no wallet extension, and no internet connection required. Everything (styling, logic, and a simulated blockchain) is inlined into the one file.

It is built as a companion to the DecentraVote pitch deck. Where the deck *describes* the architecture, this dashboard makes it tangible: you can operate a small demo election end to end and watch the smart-contract logic accept valid votes and reject invalid ones.

Because it needs no network, it is safe to demo live at a venue with unreliable or no wifi.

## Use case

Traditional and centralized electronic voting asks participants to trust a single authority and its private database: to trust that records were not altered, that no one voted twice, and that the announced tally matches the votes actually cast. That trust is hard to verify from the outside.

DecentraVote's premise is to shift that trust from *"trust the operator"* toward *"verify the record."* Voting rules live in a smart contract, and each accepted vote becomes an entry in a hash-linked public ledger that anyone can re-tally independently.

This demo exists to communicate that premise to three audiences:

- **Evaluators and judges** who want to see the concept work in seconds, not read about it.
- **Teammates and contributors** who need a shared, concrete reference for how the flow behaves.
- **Newcomers** who want an intuitive, hands-on explanation of *why* an on-chain voting record is auditable.

It is a teaching and demonstration tool for a prototype architecture — not a system for running real, binding elections.

## What it demonstrates

The dashboard has three linked tabs.

**1. Workflow.** The core loop — *Verify → Vote → Record → Verify result* — shown as four numbered steps, followed by the production system layers (React → ethers.js → Solidity → test network → audit), an explicit split of what stays **off-chain** (identity and eligibility) versus what goes **on-chain** (the vote transaction and events), and an expandable viewer showing the actual `Voting.sol` smart-contract source.

**2. Live Demo Election.** A working election you can operate:

- Connect one of several demo wallets from a voter registry.
- Cast a vote and watch it mine into a **hash-linked on-chain ledger** (a mini block explorer), complete with transaction hashes, a `VoteCast` event, gas used, and a parent-hash link to the previous block.
- See live results, turnout, and the current leader update in real time.
- Watch the on-chain guards **reject** invalid transactions — voting twice, or voting from an unregistered account — each shown with the exact revert reason.
- Run an **independent audit** that recomputes the tally purely from the recorded `VoteCast` events and confirms it matches contract state, plus a "hash chain intact" check across all blocks.

**3. Impact & Effects.** Six qualitative effects of the design (tamper-evidence, one-person-one-vote, independent verifiability, rules-enforced-in-code, transparency, privacy-aware design), a centralized-vs-DecentraVote comparison, and a small metrics panel computed live from the session you just ran.

## Quick start

1. Locate `DecentraVote-demo.html`.
2. Double-click it, or right-click → *Open with* → your browser. (You can also drag it onto an open browser window.)
3. That's it — the demo loads with the election already open and a voter connected, so you can vote with a single click.

No installation, accounts, wallet, or network connection are needed.

## Guided demo walkthrough

A complete run takes about two minutes:

1. Open the **Live Demo Election** tab. The election starts **open**, with the first eligible voter already connected.
2. In the **Ballot**, pick a candidate and click **Submit vote transaction**. A confirmation appears and a new block mines into the on-chain ledger below.
3. Pick a candidate again and submit once more as the same voter. The transaction **reverts** with `voter has already voted` — the `!hasVoted` guard firing. No block is added.
4. In the **voter registry**, connect **Guest · Unregistered**, choose a candidate, and submit. It **reverts** with `voter is not eligible` — the `isEligible` guard firing.
5. Scroll to the **On-chain ledger** and note the *"Hash chain intact ✓"* indicator; each block links to its parent by hash.
6. Open the **Independent audit** panel: the tally rebuilt from `VoteCast` events matches contract state exactly, and the rejected transactions are listed with their reasons.
7. Use **Close election** to lock results, then **Reset** to redeploy a fresh contract and start over.

Talking point: the guards are enforced in the contract logic, so the *reverts are the feature* — they show the rules holding even when someone tries to break them.

## The demo election

**Election:** *DecentraVote Demo — Student Council 2026*
**Network label:** Sepolia (an Ethereum test network) — shown for realism; see limitations below.

**Candidates**

| id | Name | List |
|----|------|------|
| 0  | Aisha Verma | Unity Front |
| 1  | Rohan Das | Progress Alliance |
| 2  | Meera Nair | Forward Bloc |
| 3  | NOTA | None of the Above |

**Voters:** six eligible voters (Aarav, Diya, Kabir, Sara, Vivaan, Anaya) registered as eligible, plus one unregistered **Guest** account used to demonstrate the eligibility guard.

**Admin controls:** open the election, close it (to lock results for audit), and reset (to redeploy a fresh contract with a new genesis block).

All names are fictional and used only for illustration.

## How it works

The page ships with a small JavaScript engine that **simulates** the blockchain layer so the flow can run with zero setup. The engine mirrors the smart-contract logic exactly:

- **Four on-chain guards** are checked, in the same order as the contract, on every vote.
- Each successful vote appends a **block** to an in-memory chain. Every block stores its own hash and its parent's hash, forming a tamper-evident chain — altering a past block breaks the link, which the "verify chain" check detects.
- Each successful vote emits a **`VoteCast` event**. The audit recomputes results *only* from these events, demonstrating that the tally is reproducible from the public record rather than taken on faith.
- Rejected attempts are recorded separately with their revert reason, so the guards are visible in action.

How each rule maps to the contract:

| Rule enforced on-chain | Solidity `require` | Revert message in the demo |
|---|---|---|
| Election must be open | `require(electionActive)` | `election is not active` |
| Voter must be registered eligible | `require(isEligible[msg.sender])` | `voter is not eligible` |
| One vote per voter | `require(!hasVoted[msg.sender])` | `voter has already voted` |
| Candidate must exist | `require(candidateId < candidates.length)` | `invalid candidate` |

The same engine is embedded verbatim inside the HTML file and also provided separately as `engine.js` so it can be unit-tested (see [Testing and QA](#testing-and-qa)).

## The smart contract

The demo represents the following Solidity contract, which you can also view from the **Workflow** tab. Identity and eligibility are handled off-chain; only the vote transaction, tallies, and events live on-chain, and no personally identifiable data is stored on the contract.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

/// @title DecentraVote — minimal on-chain voting
/// @notice Identity & eligibility are handled OFF-CHAIN. Only the vote
///         transaction and tallies live on-chain. No PII is stored here.
contract Voting {
    struct Candidate { string name; uint256 voteCount; }

    address public admin;
    bool    public electionActive;
    Candidate[] public candidates;
    mapping(address => bool) public isEligible; // set from off-chain check
    mapping(address => bool) public hasVoted;
    uint256 public totalVotes;

    event VoterRegistered(address indexed voter);
    event ElectionStatusChanged(bool active);
    event VoteCast(address indexed voter, uint256 indexed candidateId);

    modifier onlyAdmin() { require(msg.sender == admin, "not admin"); _; }

    constructor(string[] memory names) {
        admin = msg.sender;
        for (uint256 i = 0; i < names.length; i++)
            candidates.push(Candidate({ name: names[i], voteCount: 0 }));
    }

    function setElectionActive(bool active) external onlyAdmin {
        electionActive = active;
        emit ElectionStatusChanged(active);
    }

    // Register an address whose eligibility was verified off-chain.
    function registerVoter(address voter) external onlyAdmin {
        isEligible[voter] = true;
        emit VoterRegistered(voter);
    }

    // Cast one vote. The four core rules are enforced on-chain.
    function vote(uint256 candidateId) external {
        require(electionActive,                 "election is not active");
        require(isEligible[msg.sender],         "voter is not eligible");
        require(!hasVoted[msg.sender],          "voter has already voted");
        require(candidateId < candidates.length,"invalid candidate");

        hasVoted[msg.sender] = true;
        candidates[candidateId].voteCount += 1;
        totalVotes += 1;
        emit VoteCast(msg.sender, candidateId);
    }

    // Anyone can read tallies straight from chain state for audit.
    function getResult(uint256 candidateId)
        external view returns (string memory name, uint256 voteCount)
    {
        Candidate storage c = candidates[candidateId];
        return (c.name, c.voteCount);
    }
}
```

## Production stack vs this demo

The demo faithfully mirrors the production logic but runs it in the browser so it needs no setup. The intended production stack, and how each layer is represented here, is:

| Layer | Production | In this demo |
|---|---|---|
| Frontend | React.js, HTML5, CSS / Tailwind | Vanilla HTML/CSS/JS, single file |
| Web3 bridge | ethers.js + MetaMask / wallet | Simulated in-browser contract calls |
| Smart contract | Solidity (`Voting.sol`) | JS engine mirroring the same guards |
| Blockchain | Ethereum-compatible test network (e.g. Sepolia) | Simulated hash-linked blocks |
| Tooling | Hardhat, Node.js, npm | None required |

In production, the exact rules shown here run as the Solidity contract on an Ethereum-compatible test network, called from a React front end through ethers.js and a browser wallet.

## Honesty and limitations

This is a **prototype** and a **simulation**. It is deliberately transparent about what it does and does not do:

- The blockchain is **simulated in-browser**. The hashes are illustrative and deterministic — they are *not* keccak256, and there is no peer-to-peer network, consensus, mining difficulty, or real cryptographic signing.
- Identity and eligibility are treated as **off-chain** concerns. The demo does not implement a real identity/credential system; it only models whether an address was registered as eligible.
- **Ballot secrecy** is not engineered in this demo. A production design would need careful handling of vote privacy versus public verifiability.
- All numbers shown in the Impact tab are **computed live from the demo session you just ran** — they are not projected or real-world statistics.
- Candidates, voters, and the election are **fictional**, for illustration only.
- Blockchain here means **transparency, tamper-evidence, and independent verifiability** — not a claim of absolute or unbreakable security, and not a claim that blockchain is automatically superior for every real election. Real deployments still depend on a trustworthy off-chain eligibility layer, ballot-privacy design, endpoint security, and scalability/gas handling.

## Testing and QA

The correctness-critical engine ships with an automated test suite. With [Node.js](https://nodejs.org) installed:

```bash
node engine.test.js
```

This runs 34 assertions covering hash/address formats and determinism, the fresh-state setup, all four guards (inactive election, no wallet, ineligible voter, invalid candidate, double vote), the happy path (block creation, tally increment, `VoteCast` emission), the event-derived audit matching contract state, hash-chain integrity plus tamper detection, and turnout math.

The shipped demo was additionally verified to be **fully self-contained** (no external network requests of any kind) and was driven through the complete click-through flow — tab switches, vote, double-vote revert, ineligible revert, close, and reset — with no errors.

The interface respects `prefers-reduced-motion` for users who disable animations, and works in any current desktop or mobile browser.

## Files in this folder

- **`DecentraVote-demo.html`** — the interactive demo. **This is the file to open.**
- `engine.js` — the simulated voting engine (the same logic embedded in the HTML, exposed for testing).
- `engine.test.js` — the 34-assertion test suite for the engine.
- `DecentraVote.pptx` — the DecentraVote pitch deck (related project artifact).
- `README.md` — this document.

## From prototype to production

Natural next steps to take this from demonstration to a testnet deployment:

- Compile and deploy `Voting.sol` to a public test network with **Hardhat**, and replace the simulated calls with **ethers.js** reads/writes.
- Add **MetaMask / wallet** connection in place of the demo's account picker.
- Build the front end as a **React** app, reusing the same three-view structure.
- Add a real **off-chain eligibility/identity** service that gates on-chain `registerVoter` calls.
- Explore **ballot-privacy** techniques appropriate to the deployment context.

A small live enhancement worth adding for demos: a *"tamper"* control on the ledger that deliberately breaks one block's parent-hash link, so the "chain broken" detection can be shown firing on stage.

## Links

- GitHub repository: `https://github.com/apurvaanand51/decentravote`
- Live demo (hosted): `https://de-centravote.netlify.app/`
- Team / contact: Apurva Anand | Ishika Pandey | Himanshu Rai | Arjita Shrivastav | Mirza Hayat
