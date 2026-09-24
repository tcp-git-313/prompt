# W1-LAUNCHER-P1B — Launcher Seat Workbench Contract Audit

Read-only only. Inspect Launcher and Platform sources and freeze the minimum safe Workbench scope for first commercial launch of Post, DM and ORC. Confirm existing customer, member, entitlement and Seat admin RPCs; identify committed vs dirty vs live Workbench code; and design the smallest server-side member read contract that exposes a stable user_id for Seat assignment. Do not use email-position matching or heuristics.

Also freeze the minimum member lifecycle contract (add, suspend/reactivate, remove), customer lifecycle contract (create/view/update/disable/reactivate), and MUST_HAVE UI: customer search/create/detail/status, member list/add/suspend/reactivate/remove, entitlement/status/seat_limit, Seat assigned count/list/assign/release.

Return SAFE_MEMBER_ID_CONTRACT_FROZEN, MEMBER_LIFECYCLE_CONTRACT_FROZEN, CUSTOMER_LIFECYCLE_CONTRACT_FROZEN, WORKBENCH_MUST_HAVE_SCOPE_FROZEN, exact later implementation tasks, and READY_FOR_LAUNCHER_WORKBENCH_IMPLEMENTATION. Do not modify files, create RPCs, mutate Staging/Production, deploy, commit or push.