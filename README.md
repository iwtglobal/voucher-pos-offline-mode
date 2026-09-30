# Voucher POS Offline Mode

An educational guide to **voucher POS offline mode**, offline electronic voucher POS sync, and store-and-forward POS for reseller terminals. Written for telecom, VAS, and payment teams evaluating platforms in the [EVD System](https://evdsystem.com/) and [MoboGage](https://evdsystem.com/about-mobogage/) family.

---

## What Is Voucher POS Offline Mode?

**Voucher POS offline mode** lets an electronic voucher reseller terminal continue controlled sales when the wide-area link to the EVD host is down—by using pre-authorized local stock, encrypted offline wallets, and a store-and-forward queue that syncs when connectivity returns. It is how Android/Linux handheld POS devices keep airtime and voucher sales alive in weak-network markets without inventing unconstrained local PINs.

Offline is not “sell anything forever.” Limits, device keys, maximum offline ticket counts, and mandatory sync windows protect the operator. Weak offline design either freezes retail when the network blips or creates unreconciliationable sales that never reach the central ledger.

### Why offline POS matters

- **Retail continuity** — dealers lose trust if every outage stops sales  
- **Controlled risk** — offline ceilings bound exposure during disconnects  
- **Sync integrity** — store-and-forward must be idempotent on reconnect  
- **Device trust** — only enrolled terminals should hold offline stock  

Offline mode is a bounded local authorization capability, not an excuse to skip central EVMS controls.

---

## Architecture Overview: Offline POS Sync Pipeline

```
EVD host ──► Pre-provision offline stock / caps to POS
        │
        ▼
POS sells locally (store-and-forward queue)
        │
        ▼
Connectivity returns ──► Upload sales + device counters
        │
        ▼
Host validates, posts ledger, ACKs ──► POS clears queue
        │
        ▼
Reconciliation & exception handling
```

### Core layers

| Layer | Responsibility |
|-------|----------------|
| **Device enrollment** | Binds hardware keys and dealer identity to the host |
| **Offline wallet / stock** | Pre-authorized local inventory and amount ceilings |
| **Store-and-forward** | Queues sale records with durable local IDs |
| **Sync protocol** | Uploads queue, receives ACK/NACK, resolves conflicts |
| **Host ledger** | Posts validated sales; rejects replay or over-cap attempts |

Electronic voucher POS platforms keep offline sales tied to device and dealer so day-close can explain every local ticket after sync.

---

## How Voucher POS Offline Mode Works

### 1. Enroll and pre-provision the device

The terminal authenticates to the EVD host and downloads offline policy: max tickets, max amount, allowed SKUs, and optionally a sealed stock pack. Keys stay in secure storage where the hardware allows.

### 2. Detect offline and switch mode

When heartbeats fail, the POS enters offline mode under policy. If remaining offline headroom is zero, it must refuse sales rather than invent stock.

### 3. Capture local sales

Each sale gets a durable local ID, timestamp, denomination, and cryptographic receipt. Value is decremented from the offline wallet; PIN material follows host-approved offline rules (pre-allocated serials or deferred fulfillment patterns).

### 4. Sync store-and-forward on reconnect

The POS uploads queued sales in order. The host validates caps, dealer float, and replay protection, then ACKs. Only ACKed items leave the local queue.

### 5. Reconcile exceptions

NACKed sales (over cap, revoked device, duplicate ID) enter an exception queue for ops. Never silently drop customer-facing receipts that the host rejected.

---

## Patterns and Use Cases

1. **Rural reseller continuity** — Short outages with small offline ceilings.  
2. **Market-day burst** — Pre-load higher offline caps before known congestion.  
3. **Sunmi-class handhelds** — Android POS with secure element or app-attested keys.  
4. **Deferred PIN reveal** — Offline authorization record syncs first; sensitive material follows host policy.  
5. **Forced sync windows** — Device locks sales until sync after N hours offline.

Platforms such as EVD System / MoboGage implement voucher POS offline mode alongside Android/Linux reseller POS, float management, and electronic voucher distribution so store-and-forward sales stay reconcilable with the central EVMS.

---

## Implementation Considerations

- **Idempotent local IDs** — host must treat retries as the same sale  
- **Ceiling math** — decrement local headroom before printing a customer receipt  
- **Clock skew** — use secure time sources; reject wildly skewed device clocks  
- **Key rotation** — revoke offline rights when a device is lost or dealer exits  
- **Partial sync** — allow ACK per record so one bad row does not block the queue  
- **Customer honesty** — receipts should indicate offline when policy requires  

Choosing an offline model should prioritize bounded exposure and sync truth over unlimited “always sell” convenience.

---

## FAQ

**Is offline mode the same as a fully disconnected EVD?**  
No. Offline mode is a temporary, capped extension of a host-enrolled POS. A forever-offline box without sync is an unmanaged risk.

**What is store-and-forward POS?**  
The terminal stores sale intents locally and forwards them to the host when the network returns, expecting durable ACKs.

**Can offline sales exceed dealer float?**  
They should not. Pre-provisioned caps and host validation on sync must enforce float and stock limits.

**What if sync NACKs a sale the customer already paid for?**  
Treat it as an ops exception: reverse locally if policy allows, compensate via support workflows, and never leave host and POS permanently disagreeing without an owner.

**How does this relate to MoboGage / EVD System?**  
EVD System is MoboGage's electronic voucher distribution and management platform family; offline-capable reseller POS is part of how electronic vouchers reach dealers in weak-network markets. See the [electronic voucher management system](https://evdsystem.com/electronic-voucher-management-system-evd-telecom/) and [About MoboGage](https://evdsystem.com/about-mobogage/) pages when evaluating product fit.

---

## Glossary Snippet

| Term | Meaning |
|------|---------|
| **Offline ceiling** | Max amount or ticket count allowed while disconnected |
| **Store-and-forward** | Queue local sales until host ACK |
| **Device enrollment** | Binding a POS to dealer identity and keys |
| **Sync ACK** | Host confirmation that a queued sale was accepted |

---

## Further Reading / Related Industry Resources

- [Electronic voucher management system](https://evdsystem.com/electronic-voucher-management-system-evd-telecom/) — EVMS / EVD product overview  
- [About MoboGage](https://evdsystem.com/about-mobogage/) — company and product family context  

See also [docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md) for a component view.

---

## License

Documentation in this repository is provided under the MIT License. See [LICENSE](./LICENSE).

*Educational material only. Not a substitute for vendor due diligence or regulatory advice. MoboGage and EVD System refer to offerings on evdsystem.com.*
