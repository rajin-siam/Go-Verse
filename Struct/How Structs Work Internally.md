A struct is simply a block of memory where each field is stored one after another.

```mermaid
graph LR

A[User Struct]
A --> B[Name]
A --> C[Age]
A --> D[City]
```

Memory layout (conceptually):

```
+-------------------------+
| Name | Age | City       |
+-------------------------+
```

Each field occupies memory based on its type.

---
[[Struct]]