[[Packages in Go]]

Go prefers short, descriptive package names.

Good:

```
http
json
time
order
payment
user
```

Bad:

```
ordermanagementsystem
myutilities
commonfunctions
```

Package names become part of your API:

```
payment.Process()
```

reads much better than

```
paymentmanagementsystem.Process()
```