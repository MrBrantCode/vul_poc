# mall4j vulnerabilities

- Repository: https://github.com/gz-yami/mall4j
- Affected version: c69b679 (2026-11-02, current master)
- Vendor status: reported privately on 2026-11-03; awaiting vendor confirmation and fix
- Disclosure status: coordinated disclosure, 90-day window until 2027-02-01

## Vulnerability directories

- cancelled_order_pay/ - a cancelled order can still be paid and the stock is not deducted (MJ-1)
- negative_immediate_purchase/ - negative quantity in the immediate purchase path increases stock (MJ-2)
- collection_idor/ - collection page returns collections of all users (MJ-3, implementation oversight)
- delete_account_token_revocation/ - deleted member tokens remain valid and can be refreshed (MJ-4)
- concurrent_cancel_stock_refund/ - concurrent cancellation restores stock multiple times (MJ-5)

Note: MJ-6 (review ownership check) was already fixed in c69b679 and is therefore not included in this archive.
