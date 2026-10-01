# Local Security Review Kill Conditions

Stop the affected test immediately if it would require any of the following:

1. Vercel live services, production APIs, or customer deployments.
2. Accounts, teams, projects, data, or infrastructure not controlled by the
   workspace owner.
3. Real personal data, access tokens, API keys, credentials, or secrets.
4. Persistence, destructive modification, service degradation, denial of
   service, or resource exhaustion.
5. Privileged attacker access that defeats the boundary being claimed.
6. Network access from a proof of concept to a non-loopback service.
7. Testing a provider package, third-party target, demo, or intentionally unsafe
   application as though it were an eligible core SDK issue.
8. Public disclosure, report submission, or contact with Vercel or HackerOne
   without a separate instruction.

Use synthetic markers for impact. If unexpected real data or secrets appear,
stop, do not retain them, sanitize any logs, and record only that the stop
condition was triggered.
