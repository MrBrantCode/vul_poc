# newbee-mall vulnerabilities

- Repository: https://github.com/newbee-ltd/newbee-mall
- Affected version: main branch a069069 (2025-10-27, current main)
- Vendor status: reported privately on 2026-11-03; awaiting vendor confirmation and fix
- Disclosure status: coordinated disclosure, 90-day window until 2027-02-01

## Vulnerability directories

- negative_cart_quantity/ - negative cart quantity offsets the order amount and increases stock (NB-1)
- locked_member_session/ - locking a member does not invalidate the existing session (NB-2)
- multi_item_stock_race/ - multi-item stock deduction only checks one statement result and oversells (NB-3)
- concurrent_cancel_stock_refund/ - concurrent cancellation restores stock multiple times (NB-4)
