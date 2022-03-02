## 0.8.2

Low level inliner

If code inside of function is simple enough and doesn’t cause side effects, the compiler inlines the entire function and if there are no additional references to the function then the tag to the function is removed entirely

```solidity
PUSH <tag>
JUMP
...
<tag>:
[ROUTINE]
JUMP
```

```solidity
[ROUTINE]
JUMP
...
<tag>:
[ROUTINE]
JUMP
```