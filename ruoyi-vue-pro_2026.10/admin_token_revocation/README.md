# Admin account invalidation does not revoke tokens (YD-2)

## Summary

When an admin account is deleted, batch deleted, or disabled through the Excel import, the application does not revoke the OAuth2 access token and refresh token that were already issued to that account. The old tokens remain usable, and the refresh token can still be exchanged for a new access token after the account has been invalidated.

- Affected version: ruoyi-vue-pro master branch before 2026-11-05 (reported at commit c799e27c54)
- Prerequisite: the account owner holds a token that has not expired
- Impact: a deleted or disabled admin account can keep operating the admin API with its previous permissions

## Root Cause

The disable path was fixed previously: AdminUserServiceImpl#updateUserStatus calls oauth2TokenService.removeAccessToken when the status is changed to disabled. The other account invalidation paths in the same class do not remove tokens:

- deleteUser
- deleteUserList
- importUserList when the imported status is disabled

TokenAuthenticationFilter only validates the token itself and does not check whether the user still exists or is enabled. OAuth2TokenServiceImpl#refreshAccessToken also does not check the account status before issuing a new token.

Code references (c799e27c54):

- https://github.com/YunaiV/ruoyi-vue-pro/blob/c799e27c54/yudao-module-system/src/main/java/cn/iocoder/yudao/module/system/service/user/AdminUserServiceImpl.java#L258-L276 (disable path, removes token)
- https://github.com/YunaiV/ruoyi-vue-pro/blob/c799e27c54/yudao-module-system/src/main/java/cn/iocoder/yudao/module/system/service/user/AdminUserServiceImpl.java#L280-L299 (delete user)
- https://github.com/YunaiV/ruoyi-vue-pro/blob/c799e27c54/yudao-module-system/src/main/java/cn/iocoder/yudao/module/system/service/user/AdminUserServiceImpl.java#L300-L313 (batch delete)
- https://github.com/YunaiV/ruoyi-vue-pro/blob/c799e27c54/yudao-module-system/src/main/java/cn/iocoder/yudao/module/system/service/user/AdminUserServiceImpl.java#L590-L608 (Excel import)
- https://github.com/YunaiV/ruoyi-vue-pro/blob/c799e27c54/yudao-module-system/src/main/java/cn/iocoder/yudao/module/system/service/oauth2/OAuth2TokenServiceImpl.java#L73-L100 (refresh token)
- https://github.com/YunaiV/ruoyi-vue-pro/blob/c799e27c54/yudao-framework/yudao-spring-boot-starter-security/src/main/java/cn/iocoder/yudao/framework/security/core/filter/TokenAuthenticationFilter.java#L71-L92 (token validation)

## Reproduction

1. An administrator creates an admin account A with a normal role. A logs in and stores its accessToken and refreshToken.
2. The administrator disables A through Excel import (or an equivalent status update).
3. A uses the old accessToken to call an admin API. The request succeeds.
4. The refreshToken can still be used to obtain a new accessToken.
5. Control: disabling the same account through the disable button (PUT /admin-api/system/user/update-status) invalidates the old token immediately and returns 401.

## Observed Result

- Before disabling: GET /admin-api/system/user/get?id=1 returns code=0.
- After disabling through Excel import: the same old accessToken returns code=0 and admin user data; the refresh token still returns code=0 and issues a new access token.
- After deleting the user: the old accessToken still passes authentication for /admin-api/system/auth/get-permission-info (HTTP 200, code=0); permission-protected endpoints return 403 because the role relations were deleted. The refresh token is still accepted and fails later with an NPE when the user is loaded.
- Control path: after disabling through the disable button, the same old accessToken returns 401 / account not logged in.

Raw request and response examples:

    # After Excel-import disable: old token still works
    GET /admin-api/system/user/get?id=1
    -> 200 {"code":0, ...admin user data...}

    # After user deletion: old token still passes authentication
    GET /admin-api/system/auth/get-permission-info
    -> 200 {"code":0,"data":null}

    # Refresh after deletion: accepted, then fails while loading the user
    POST /admin-api/system/auth/refresh-token?refreshToken=<old>
    -> 500 (NPE, user does not exist)

    # Control: disable button path
    PUT /admin-api/system/user/update-status {"id":<id>,"status":0}
    GET /admin-api/system/user/get?id=1
    -> 401 {"code":401,"msg":"account not logged in"}

## Impact

- A deleted or disabled admin account can continue to use the admin API with its old permissions until the token expires.
- The refresh token allows the session to be extended even after the account has been invalidated.
- The Excel import disable path keeps the role relations, so the old token can still access all endpoints allowed by the original role.

## Remediation

1. In deleteUser, deleteUserList and the disabled branch of importUserList, call oauth2TokenService.removeAccessToken(userId, UserTypeEnum.ADMIN) before the account becomes invalid.
2. As defence in depth, check that the user still exists and is enabled during token validation and token refresh.
3. In refreshAccessToken, validate the account status before issuing a new token and return a clear error instead of an NPE.

## References

- Fix commit: https://github.com/YunaiV/ruoyi-vue-pro/commit/e09a9c20663465970705c1ae44f7313491cfc24a (master-jdk17, 2026-11-05)
- Vendor confirmation: https://github.com/MrBrantCode/vul_poc/blob/main/ruoyi-vue-pro_2026.10/Disclosure_Process.md
