[[Packages in Go]]

Coupling measures how much one package depends on another.

High coupling:

```
Order

↓

Payment

↓

Inventory

↓

Shipping

↓

Notification

↓

Analytics
```

A change in one package can ripple through many others.

Good Go design aims for low coupling, where packages interact through well-defined interfaces.