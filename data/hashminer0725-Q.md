 ## 1. Unnecessary `require` check.

```
require(true, "unable to find released NFT in ManagedNFTs");
```

This require is passed even if the NFT releasing is failed before.
