## Compensating Transactions with Sagas

If a local transaction fails, the saga executes a series of compensating transactions that undo the changes that were made by the preceding local transactions. In Saga patterns: Compensable transactions are transactions that can potentially be reversed by processing another transaction with the opposite effect.

- More - https://microservices.io/patterns/data/saga.html
