
## Principles of Channel ownership
1. Single writer principles
2. 


## Common Mistakes to Avoid
1. Closing a channel more than once (causes panic).
2. Sending on a closed channel (causes panic).
3. Closing from the receiver side

## Best Practices:
1. Establish clear ownership of channel closing.
2. Consider leaving channels open if ownership is unclear
3. Use done channels or context for cancellation.
4. Document channel closing responsibility explicitly in complex scenarios.

## 