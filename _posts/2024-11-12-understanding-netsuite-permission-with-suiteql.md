---
title: "Understanding NetSuite Permissions with SuiteQL"
date: 2024-11-12
categories:
  - NetSuite
  - Development
tags:
  - NetSuite
  - SuiteQL
  - Permissions
  - Database
  - Security
---

# Understanding NetSuite Permissions with SuiteQL

As NetSuite developers, understanding and managing permissions is crucial for building secure and robust applications. In this post, I'll share some useful SuiteQL queries for analyzing and troubleshooting role permissions in NetSuite.

## Basic Permission Check Query

Let's start with a basic query that shows role permissions and their levels:

```sql
SELECT
    rp.role as user_role,
    rp.permkey,
    rp.permlevel as perm_actual,
    r.name as role_name,
    CASE
        WHEN rp.permlevel = 0 THEN 'None'
        WHEN rp.permlevel = 1 THEN 'View'
        WHEN rp.permlevel = 2 THEN 'Create'
        WHEN rp.permlevel = 3 THEN 'Edit'
        WHEN rp.permlevel = 4 THEN 'Full'
    END as permission_level
FROM
    rolepermissions rp
    JOIN role r ON r.id = rp.role
WHERE
    rp.role = 3  -- Replace with specific role ID
    AND rp.permkey IN (
        'TRAN_FIND',
        'TRAN_SALESORD',
        'ADMI_ACCOUNTING',
        'ADMI_MANAGE_LICENCES'
    )
ORDER BY
    rp.permkey;
```

This query provides:

- User role ID and name
- Permission keys
- Actual permission levels
- Human-readable permission levels

## Extended Permission Analysis

For more detailed analysis, we can use this extended query that includes permission status:

```sql
SELECT
    rp.role as user_role,
    r.name as role_name,
    rp.permkey,
    rp.permlevel as perm_actual,
    CASE
        WHEN rp.permlevel = 0 THEN 'None'
        WHEN rp.permlevel = 1 THEN 'View'
        WHEN rp.permlevel = 2 THEN 'Create'
        WHEN rp.permlevel = 3 THEN 'Edit'
        WHEN rp.permlevel = 4 THEN 'Full'
    END as permission_level,
    CASE
        WHEN rp.permlevel >= 3 THEN 'Sufficient'
        ELSE 'Insufficient'
    END as permission_status
FROM
    rolepermissions rp
    JOIN role r ON r.id = rp.role
WHERE
    rp.permkey IN (
        'TRAN_FIND',
        'TRAN_SALESORD',
        'ADMI_ACCOUNTING',
        'ADMI_MANAGE_LICENCES'
    )
ORDER BY
    rp.role,
    rp.permkey;
```

## Understanding the Query Components

### Tables Used

- `rolepermissions`: Contains the mapping between roles and permissions
- `role`: Contains role definitions and names

### Key Fields

- `role`: The role ID
- `permkey`: The permission identifier
- `permlevel`: The numeric permission level (0-4)

### Permission Levels

1. None (0): No access
2. View (1): Read-only access
3. Create (2): Can create new records
4. Edit (3): Can modify existing records
5. Full (4): Complete access

## Common Use Cases

### Checking Specific Role Permissions

Add this filter to check a specific role:

```sql
AND rp.role = 3  -- Replace with your role ID
```

### Filtering by Permission Level

To find all permissions at or above a certain level:

```sql
AND rp.permlevel >= 3  -- Minimum permission level
```

### Searching by Role Name

To filter roles by name pattern:

```sql
AND r.name LIKE '%Admin%'  -- Role name filter
```

## Best Practices

1. **Always Join with Role Table**

   - Provides readable role names
   - Ensures role exists

2. **Use Meaningful Aliases**

   - Makes queries more readable
   - Clarifies output

3. **Include CASE Statements**

   - Converts numeric levels to readable text
   - Adds meaningful status indicators

4. **Order Results**

   - Makes output easier to analyze
   - Groups related permissions together

## Conclusion

These SuiteQL queries provide powerful tools for analyzing and troubleshooting NetSuite permissions. They can help you:

- Audit role permissions
- Debug access issues
- Document role configurations
- Validate permission settings

Remember to always consider security implications when querying permission data and restrict access to these queries appropriately.

## Next Steps

In future posts, I'll cover:

- Dynamic permission checking in SuiteScript
- Permission optimization patterns
- Role inheritance and management
- Custom permission schemes

Stay tuned for more NetSuite development insights!
