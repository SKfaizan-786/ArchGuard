---
id: "ADR-005"
code: "ADR-005"
title: "Mandate Idempotency Keys on Payment Ingress Endpoints"
status: "ACCEPTED"
rule_type: "ENDPOINT_EXECUTION_MODE"
target_property: "execution_mode"
expected_value: "idempotent_keyed"
governs_target: "/v1/webhooks/payment"
author_handle: "sam_platform"
---

# ADR-005: Mandate Idempotency Keys on Payment Ingress Endpoints

## Status
ACCEPTED (Active & Enforced)

## Context
Payment webhook duplication can cause double-charging or duplicate credits.

## Decision
All payment ingress endpoints MUST validate RFC-compliant idempotency tokens.
