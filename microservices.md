# Microservices

1. Each service has their own datastore
1. Each service is 100% responsible for their datastore
1. Each service must never access another service's datastore directly
1. Each service is an isolated silo
1. We don't want one service to bring down another service in ways it is not expecting
