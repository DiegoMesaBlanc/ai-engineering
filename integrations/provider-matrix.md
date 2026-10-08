# Provider Matrix

| Capability | Provider | Mechanism | License / Source | Cost | Status |
|---|---|---|---|---|---|
| Git | GitHub | MCP | MIT / Open Source | Free* | Candidate |
| Git | GitLab | MCP | Platform-provided | Free tier* | Candidate |
| Git | Azure Repos | Azure DevOps MCP | Microsoft | Free MCP* | Candidate |
| Task | Azure DevOps | MCP | Microsoft | Free MCP* | Candidate |
| Task | Jira | Atlassian MCP | Apache-licensed repo | Atlassian Cloud required | Optional |
| Design | Penpot | MCP | MPL-2.0 / Open Source | Free / Self-hostable | Preferred |
| Design | Figma | MCP | Hosted integration | Depends on Figma plan | Optional |
| Design | Generic files | Local | Local | Free | Core |

## Notes

\* "Free MCP" does not mean the underlying platform is necessarily free.

The Engineering System must prefer providers that can be used without a
mandatory paid subscription.

When a platform has a free tier, evaluate its actual limits before making
it a recommended dependency.