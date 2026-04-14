# Admission Controllers

## What are Admission Controllers?

- Admission Controllers are the plugins that act as a gatekeeper for the API server.
- Admission controllers intercept requests to create, update, or delete resources after the user is authenticated and authorized, but before the object is saved to the etcd database.

### The Two Phases of Admission

- The Admission control happens in a specific, two-stage sequence to ensure that any modifications are properly validated
