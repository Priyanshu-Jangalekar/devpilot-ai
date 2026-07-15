# Coding Rules

These rules guide AI-generated code for DevPilot AI. They are principles, not restrictions. AI should use engineering judgment while following these guidelines.

## General Principles

Prioritize:

1. Readability
2. Simplicity
3. Maintainability
4. Consistency
5. Correctness

Avoid unnecessary complexity.

## Code Style

- Write code that another developer can understand quickly.
- Prefer descriptive names.
- Avoid abbreviations unless they are widely accepted.
- Give functions a single responsibility.
- Keep files focused on one purpose.

## Reuse

Before creating a component, service, utility, or hook, check whether one already exists. Prefer reusing existing code and avoid duplicate logic.

## Modularity

- Organize code around features.
- Keep modules independent.
- Minimize coupling.
- Maximize cohesion.

## Type Safety

- Prefer strong typing.
- Avoid `any` whenever possible.
- Use shared types for communication between frontend and backend.

## Error Handling

- Handle errors gracefully.
- Provide meaningful messages.
- Never silently ignore failures.

## Logging

- Log information that helps debugging.
- Do not expose sensitive data.
- Use structured logs where appropriate.

## Testing

- Include appropriate tests for new functionality.
- Prioritize testing business logic.
- Prefer meaningful tests over high coverage numbers.

## Documentation

Code should be self-explanatory. Document complex algorithms, architectural decisions, and public APIs. Avoid comments that only restate obvious code.

## Performance

Write clean code first. Optimize only after identifying a real bottleneck, and avoid premature optimization.

## Security

- Never expose secrets.
- Validate all external input.
- Use parameterized database queries.
- Follow authentication and authorization rules.

## Refactoring

Leave the codebase better than you found it. Small, continuous improvements are encouraged. Large rewrites require architectural justification.

## AI Decision Making

AI is encouraged to make implementation decisions when:

- Requirements are clear.
- Architecture is respected.
- Consistency is maintained.

AI should ask for clarification when:

- Requirements conflict.
- Architecture is unclear.
- Security may be affected.
- Data models require changes.
- Business logic is ambiguous.

## Final Principle

Write code as if another engineer will maintain it for the next five years.
