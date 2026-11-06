# VulDB submission drafts (4 ruoyi-vue-pro vulnerabilities)

Submission entry: https://vuldb.com/?submit (login required; form is typically at https://vuldb.com/vuln/add)
Submit one entry per vulnerability. Do not merge multiple vulnerabilities into one entry.
Descriptions are in English. CVSS vectors are suggestions and can be adjusted with the FIRST calculator.

## YD-1 Negative order quantity

- Vendor: YunaiV
- Product: ruoyi-vue-pro
- Version: master before 2026-11-05 (e.g. c799e27c54)
- Class: Input Validation
- CWE: CWE-20
- CVSS: CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:N/I:H/A:H (8.1)
- Fixed: https://github.com/YunaiV/ruoyi-vue-pro/commit/64d68433ed77ccdde6b1652845d2d1efc1d08ca7 (master-jdk17)
- Advisory URL: https://github.com/MrBrantCode/vul_poc/tree/main/ruoyi-vue-pro_2026.10/negative_order_quantity
- Description:
  A vulnerability was found in YunaiV ruoyi-vue-pro. The order settlement request does not validate the quantity of an order item. A remote authenticated member can submit a negative product count; the negative subtotal offsets other items and the stock update increases the stock. This allows purchasing expensive goods at a very low price and causes stock inflation. The vendor confirmed the issue and fixed it in commit 64d68433ed77ccdde6b1652845d2d1efc1d08ca7 on branch master-jdk17 (default master branch sync pending at the time of this report).

## YD-2 Admin token revocation

- Vendor: YunaiV
- Product: ruoyi-vue-pro
- Version: master before 2026-11-05 (e.g. c799e27c54)
- Class: Insufficient Session Expiration
- CWE: CWE-613
- CVSS: CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:N (8.1)
- Fixed: https://github.com/YunaiV/ruoyi-vue-pro/commit/e09a9c20663465970705c1ae44f7313491cfc24a (master-jdk17)
- Advisory URL: https://github.com/MrBrantCode/vul_poc/tree/main/ruoyi-vue-pro_2026.10/admin_token_revocation
- Description:
  A vulnerability was found in YunaiV ruoyi-vue-pro. Deleting an admin account, batch deleting admin accounts, or disabling an admin account via Excel import does not revoke already issued OAuth2 access tokens or refresh tokens. A previously issued token remains usable and can be refreshed after the account has been invalidated. The vendor confirmed and fixed the issue by invalidating access tokens, refresh tokens and authorization codes in the same transaction and clearing the related Redis cache (commit e09a9c20663465970705c1ae44f7313491cfc24a on master-jdk17).

## YD-3 Brokerage withdrawal race

- Vendor: YunaiV
- Product: ruoyi-vue-pro
- Version: master before 2026-11-05 (e.g. c799e27c54)
- Class: Race Condition
- CWE: CWE-362
- CVSS: CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:N/I:H/A:N (6.5)
- Fixed: https://github.com/YunaiV/ruoyi-vue-pro/commit/64d68433ed77ccdde6b1652845d2d1efc1d08ca7 (master-jdk17)
- Advisory URL: https://github.com/MrBrantCode/vul_poc/tree/main/ruoyi-vue-pro_2026.10/brokerage_withdrawal_race
- Description:
  A vulnerability was found in YunaiV ruoyi-vue-pro. The brokerage withdrawal logic reads the user balance and then deducts it without an atomic balance lower-bound condition. Concurrent withdrawal requests can therefore drive the brokerage balance below zero. The vendor confirmed and fixed the issue by using an atomic conditional deduction; requests with insufficient balance now fail and roll back (commit 64d68433ed77ccdde6b1652845d2d1efc1d08ca7 on master-jdk17).

## YD-4 Member point deduction race

- Vendor: YunaiV
- Product: ruoyi-vue-pro
- Version: master before 2026-11-05 (e.g. c799e27c54)
- Class: Race Condition
- CWE: CWE-362
- CVSS: CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:N/I:H/A:N (6.5)
- Fixed: https://github.com/YunaiV/ruoyi-vue-pro/commit/64d68433ed77ccdde6b1652845d2d1efc1d08ca7 (master-jdk17)
- Advisory URL: https://github.com/MrBrantCode/vul_poc/tree/main/ruoyi-vue-pro_2026.10/member_point_deduction_race
- Description:
  A vulnerability was found in YunaiV ruoyi-vue-pro. Member points are deducted after an order is created; when the point deduction fails because of insufficient balance, the code only logs the error and returns without failing the order. Concurrent orders can therefore reuse the same points. The vendor reproduced the issue under MySQL READ COMMITTED and fixed it by rolling back the order transaction when the point deduction fails (commit 64d68433ed77ccdde6b1652845d2d1efc1d08ca7 on master-jdk17).
