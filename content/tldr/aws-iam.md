---
date: '2025-08-22T09:46:49+05:30'
draft: false
title: 'AWS IAM Policy'
tags: ["AWS", "IAM", "json"]
---

```json
{
  "Version": "2012-10-17",      // Required: Policy language version (always use latest)
  "Id": "PolicyIdentifier",     // Optional: Unique identifier for the policy
  "Statement": [                // Required: One or more statements
    {
      "Sid": "StatementId",     // Optional: Identifier for the statement
      "Effect": "Allow",        // Required: Either "Allow" or "Deny"
      "Principal": {            // Optional: Defines who the policy applies to
        "AWS": "arn:aws:iam::123456789012:role/MyRole"
      },
      "Action": [               // Required: List of actions (API calls)
        "s3:GetObject",
        "s3:PutObject"
      ],
      "NotAction": "s3:DeleteObject", // Optional: Defines actions explicitly excluded
      "Resource": [             // Required (except for some policies): Target resources
        "arn:aws:s3:::my-bucket/*"
      ],
      "NotResource": "arn:aws:s3:::restricted-bucket/*", // Optional: Resources excluded
      "Condition": {            // Optional: Context-based restrictions
        "IpAddress": {
          "aws:SourceIp": "0.0.0.0/0"
        }
      }
    }
  ]
}
```
