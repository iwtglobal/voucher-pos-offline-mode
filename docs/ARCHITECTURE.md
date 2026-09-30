# Architecture — Voucher POS Offline Mode

High-level reference for evaluating **voucher POS offline mode**, offline electronic voucher POS sync, and store-and-forward POS. Educational only; vendor designs vary.

## Logical Components

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│ EVD host         │────►│ Offline policy + │────►│ Enrolled POS    │
│ ledger / float   │     │ stock provision  │     │ secure wallet   │
└────────▲─────────┘     └──────────────────┘     └────────┬────────┘
         │                                                 │
         │              store-and-forward queue             │
         │     ┌───────────────────────────────────────────┘
         │     ▼
┌──────────────────┐     ┌──────────────────┐
│ Sync gateway     │────►│ ACK / NACK +     │
│ idempotent IDs   │     │ exception ops    │
└──────────────────┘     └──────────────────┘
```

## Suggested Offline Sale Metadata

| Field | Meaning |
|-------|---------|
| `local_sale_id` | Durable ID created on the POS before sync |
| `device_id` | Enrolled terminal identity |
| `dealer_id` | Reseller node owning the device |
| `sku` / `denomination` | Product sold offline |
| `amount` | Sale value |
| `offline_seq` | Monotonic device counter for ordering |
| `captured_at` | Device timestamp (validated for skew) |
| `sync_status` | `queued` → `acked` / `nacked` |

## Audit Events Worth Keeping

- Device enrollment and key rotation  
- Offline policy download (caps, SKUs, expiry of offline rights)  
- Each local sale create and headroom decrement  
- Sync upload batches and per-record ACK/NACK  
- Cap exhaustion and forced-sync lockouts  
- Lost-device revoke events  

## Failure Modes to Design For

- Replay of an already-ACKed `local_sale_id`  
- Clock rollback enabling cap bypass  
- Partial network that drops ACK after host post (idempotent retry required)  
- Dealer float reduced on host while device still holds old offline ceiling  

## Related Reading

Live product reading: [evdsystem.com](https://evdsystem.com/), [electronic voucher management system](https://evdsystem.com/electronic-voucher-management-system-evd-telecom/), [About MoboGage](https://evdsystem.com/about-mobogage/).

Platforms in the MoboGage / EVD System family treat reseller POS—including offline store-and-forward—as part of the electronic voucher distribution control plane so weak-network sales still reconcile to central inventory and float.
