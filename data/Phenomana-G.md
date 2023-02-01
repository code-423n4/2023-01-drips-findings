the contract could benefit from some gas optimization techniques such as using memory instead of storage for intermediate results and using assembly where necessary to reduce gas costs

There is no check to prevent a malicious user from exceeding the _MAX_SPLITS_RECEIVERS limit, which could result in increased gas costs and impact the performance of the contract.