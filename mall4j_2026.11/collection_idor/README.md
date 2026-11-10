# Collection page returns collections of all users (MJ-3)

## Summary

The collection pagination endpoint returns collections from all users because the mapper query does not use the userId parameter. Any authenticated member can page through other users' collections and learn which products they collected and when.

- Affected version: mall4j current master c69b679 (2026-11-02)
- Endpoint: GET /p/user/collection/page
- Prerequisite: any authenticated member
- Impact: low; disclosure of other users' collection data

## Root Cause

UserCollectionMapper.xml:19-27 is missing a WHERE condition on the current user; the userId argument is not referenced in the SQL. The mapper interface also does not bind the parameter with @Param.

Code references (c69b679):

- https://github.com/gz-yami/mall4j/blob/c69b679/yami-shop-service/src/main/resources/mapper/UserCollectionMapper.xml#L19-L27

## Reproduction

1. Log in as any member.
2. Call GET /p/user/collection/page and page through the result.
3. The response contains collection records that belong to other users.

## Impact

- Leaks collected products and collection timestamps of other users.
- Low severity, but it is a clear implementation oversight and an access-control issue.

## Remediation

Add where uc.user_id = #{userId} to the mapper SQL and bind the mapper parameter with @Param.

## References

- Fix status: pending vendor fix
- Report: this directory
