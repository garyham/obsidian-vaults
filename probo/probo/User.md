A User is the internal respresentation of a person using the service.  Each person has a single User record.

```typescript
interface User {
	id: string
	name?: string
}
```

A User may have one or more AuthXXX (Email, OAuth) record.