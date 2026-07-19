**Composition is a programming technique where a struct or object contains other structs or objects as fields, allowing them to work together to achieve a particular functionality. It implements has-a relationship between objects .**

**In contrast, inheritance is a mechanism where a new class is derived from an existing class, inheriting all of its properties and behaviors. It implements is-a relationship between objects .**

Go prefers Composition over Inheritance.

A simple Example idea:

```
Order
  ├── HAS-A Payment → handles charging
  └── HAS-A Invoice → handles receipt
```

In Go:

```go
package main

import "fmt"

type Payment struct{}

func (p Payment) Process(amount float64) {
	fmt.Printf("Payment of $%.2f processed\n", amount)
}

type Invoice struct{}

func (i Invoice) Generate(orderID string) {
	fmt.Printf("Invoice generated for order %s\n", orderID)
}

// Order HAS-A Payment and Invoice
type Order struct {
	payment Payment
	invoice Invoice
}

func (o Order) Place(orderID string, amount float64) {
	o.payment.Process(amount)
	o.invoice.Generate(orderID)
	fmt.Println("Order placed successfully")
}

func main() {
	order := Order{
		payment: Payment{},
		invoice: Invoice{},
	}

	order.Place("ORD-101", 59.99)
}
```

---

## How composition is useful?

To understand exact how composition is useful we need to imagine a world where there is no Composition principles. So lets dive into a what if world where there is no Composition principles.

We use inheritance to extend class capabilities. Suppose we create a interface. We use interface to achieve lose coupling. But we are never sure that our interface will be same as always.

For example,

1. Method signature might change.
2. Our return type might also change.
3. We might need to add a method in our interface.

So as we can see if change in interface happens we need to fix those changes in child classes too. As we are using interface to achieve loss coupling, there might be some cases to concrete class might not need all the methods in the interface to be implemented.

We need to think a lot. We need to analyze and design the interface in such a way that it never changes. If it changes it will have repel effect across the code-base. That's why need to follow SOLID principles. But as code-bases grows these principles become very hard to be maintained. So as we can see the main problem was hierarchy based inheritance hit a bottleneck. So we need a new way to solve these problems. How we can solve the problem. That’s where composition based inheritance kicks in.

You see in composition based inheritance we will inheriting behavior from other classes through focused classes.  
So we are migrating from IS-A relationship to HAS-A relationship

Lets give a example.

```jsx
type EmailClient interface {
	SendEmail(to string, subject string, body string) error
	GetInbox() ([]string, error)
	DeleteEmail(emailID string) error
	CreateFolder(folderName string) error
}

```

Lets create 2 EmailClient

```jsx
type GmailClient struct {
	accessToken string
}

func (g GmailClient) SendEmail(to string, subject string, body string) error {
	fmt.Printf("Gmail: sending email to %s | subject: %s\n", to, subject)
	return nil
}

func (g GmailClient) GetInbox() ([]string, error) {
	return []string{"email_001", "email_002", "email_003"}, nil
}

func (g GmailClient) DeleteEmail(emailID string) error {
	fmt.Printf("Gmail: deleting email %s\n", emailID)
	return nil
}

func (g GmailClient) CreateFolder(folderName string) error {
	fmt.Printf("Gmail: creating folder '%s'\n", folderName)
	return nil
}
```

```go
type OutlookClient struct {
	accessToken string
}

func (o OutlookClient) SendEmail(to string, subject string, body string) error {
	fmt.Printf("Outlook: sending email to %s | subject: %s\n", to, subject)
	return nil
}

func (o OutlookClient) GetInbox() ([]string, error) {
	return []string{"email_101", "email_102"}, nil
}

func (o OutlookClient) DeleteEmail(emailID string) error {
	fmt.Printf("Outlook: deleting email %s\n", emailID)
	return nil
}

func (o OutlookClient) CreateFolder(folderName string) error {
	fmt.Printf("Outlook: creating folder '%s'\n", folderName)
	return nil
}

```

![image.png](attachment:b2962507-5232-4487-bde7-35d6aeb8a28b:image.png)

We might think in this way that, GmailClient and OutLookClient might need all methods which are implemented by EmailClient.

Later We Add SendGridCliend

SendGridCleintonly needs SentEmail.

But inheritance/interface contract forces ALL methods.

```go
type SendGridClient struct {
	apiKey string
}

func (s SendGridClient) SendEmail(to string, subject string, body string) error {
	fmt.Printf("SendGrid: sending email to %s | subject: %s\n", to, subject)
	return nil
}

// SendGrid has no inbox concept — but the interface forces us to implement this
func (s SendGridClient) GetInbox() ([]string, error) {
	// We have no choice. We must write something here.
	panic("SendGrid does not support inbox — it is a delivery service, not a mailbox")
}

// SendGrid has no delete concept either
func (s SendGridClient) DeleteEmail(emailID string) error {
	panic("SendGrid does not support email deletion")
}

// SendGrid has no folder concept
func (s SendGridClient) CreateFolder(folderName string) error {
	panic("SendGrid does not support folders")
}
```

```go
type EmailService struct {
	client EmailClient
}

func (es *EmailService) SendWelcomeEmail(userEmail string) error {
	return es.client.SendEmail(
		userEmail,
		"Welcome to MapNests!",
		"Thank you for joining MapNests. Start exploring indoor maps today.",
	)
}

func (es *EmailService) ListInbox() ([]string, error) {
	return es.client.GetInbox()
}

func (es *EmailService) RemoveEmail(emailID string) error {
	return es.client.DeleteEmail(emailID)
}

func (es *EmailService) OrganizeFolder(folderName string) error {
	return es.client.CreateFolder(folderName)
}
```

```go
func main() {
	fmt.Println("=== Gmail ===")
	gmailService := &EmailService{
		client: GmailClient{accessToken: "gmail_token_abc"},
	}
	gmailService.SendWelcomeEmail("rajin@gmail.com")
	emails, _ := gmailService.ListInbox()
	fmt.Println("Gmail inbox:", emails)
	gmailService.RemoveEmail("email_001")
	gmailService.OrganizeFolder("MapNests Notifications")

	fmt.Println()
	fmt.Println("=== Outlook ===")
	outlookService := &EmailService{
		client: OutlookClient{accessToken: "outlook_token_xyz"},
	}
	outlookService.SendWelcomeEmail("rajin@outlook.com")
	emails, _ = outlookService.ListInbox()
	fmt.Println("Outlook inbox:", emails)
	outlookService.RemoveEmail("email_101")
	outlookService.OrganizeFolder("Work Emails")

	fmt.Println()
	fmt.Println("=== SendGrid ===")
	sendGridService := &EmailService{
		client: SendGridClient{apiKey: "sg_key_abc123"},
	}

	// This works fine — SendGrid can send emails
	sendGridService.SendWelcomeEmail("rajin@technonext.com")

	// Now a developer on your team innocently calls this
	// They see EmailService has ListInbox() — looks safe, right?
	// The compiler gave ZERO warning. Everything compiled perfectly.
	// But at runtime — PANIC.
	fmt.Println("Calling ListInbox on SendGrid...")
	emails, _ = sendGridService.ListInbox() // PANIC here
	fmt.Println("SendGrid inbox:", emails)
}
```

So we can see that we are violating LSP from SOLID principles. Or suppose we add a method in the EmailClient interface in the future. We will go through much more change. But real codebase will be much more bigger. The problem will be much more complex. Or suppose we might need to change a method signature or return type in EmailClient. This thing will have a ripple effect in SendGrid Client.

This leads us to tight coupling. Besides if we add a method in common interface in future this will be a chaos for the methods in concrete classes.

So we need a better solution. We can use composition here to solve this problem.

### Composition-Based Design

Instead of forcing one giant contract:

Create small focused behaviors.

Small Behaviors

```jsx
type EmailSender interface {
	SendEmail(to string, subject string, body string) error
}

type InboxReader interface {
	GetInbox() ([]string, error)
}

type MailboxManager interface {
	DeleteEmail(emailID string) error
	CreateFolder(folderName string) error
}

```

FullEmailClient composes all three ? for providers that support everything.  
This is the composition step: one interface embedding three smaller ones.

![image.png](attachment:1478ba53-2207-4c9e-abc8-a479160473a0:image.png)

```go
type FullEmailClient interface {
	EmailSender
	InboxReader
	MailboxManager
}
```

```go
type GmailClient struct{ accessToken string }

func (g GmailClient) SendEmail(to, subject, body string) error {
	fmt.Printf("Gmail: sending to %s | subject: %s\n", to, subject)
	return nil
}
func (g GmailClient) GetInbox() ([]string, error) {
	return []string{"email_001", "email_002"}, nil
}
func (g GmailClient) DeleteEmail(id string) error {
	fmt.Printf("Gmail: deleting %s\n", id)
	return nil
}
func (g GmailClient) CreateFolder(name string) error {
	fmt.Printf("Gmail: creating folder '%s'\n", name)
	return nil
}
```

```go
type OutlookClient struct{ accessToken string }

func (o OutlookClient) SendEmail(to, subject, body string) error {
	fmt.Printf("Outlook: sending to %s | subject: %s\n", to, subject)
	return nil
}
func (o OutlookClient) GetInbox() ([]string, error) {
	return []string{"email_101", "email_102"}, nil
}
func (o OutlookClient) DeleteEmail(id string) error {
	fmt.Printf("Outlook: deleting %s\n", id)
	return nil
}
func (o OutlookClient) CreateFolder(name string) error {
	fmt.Printf("Outlook: creating folder '%s'\n", name)
	return nil
}

```

### SendGridClient will only implement EmailSender or embed EmailSender behavior

```go
type SendGridClient struct{ apiKey string }

func (s SendGridClient) SendEmail(to, subject, body string) error {
	fmt.Printf("SendGrid: sending to %s | subject: %s\n", to, subject)
	return nil
}

```

### **EmailService**

Will uses FullEmailClient. Which can use all Functionality.

```go
type EmailService struct {
	client FullEmailClient
}

func (es *EmailService) SendWelcomeEmail(userEmail string) error {
	return es.client.SendEmail(userEmail, "Welcome!", "Thanks for joining.")
}
func (es *EmailService) ListInbox() ([]string, error) { 
	return es.client.GetInbox() 
}
func (es *EmailService) RemoveEmail(id string) error   { 
	return es.client.DeleteEmail(id) 
}
func (es *EmailService) OrganizeFolder(name string) error {
 return es.client.CreateFolder(name) 
}

```

## SenderService

```go
type SenderService struct {
	client EmailSender
}

func (ss *SenderService) SendWelcomeEmail(userEmail string) error {
	return ss.client.SendEmail(userEmail, "Welcome!", "Thanks for joining.")
}

```

## Main Function

```jsx
func main() {
	fmt.Println("=== Gmail (full mailbox) ===")
	gmailService := &EmailService{client: GmailClient{accessToken: "gmail_token"}}
	gmailService.SendWelcomeEmail("rajin@gmail.com")
	emails, _ := gmailService.ListInbox()
	fmt.Println("Inbox:", emails)
	gmailService.RemoveEmail("email_001")
	gmailService.OrganizeFolder("MapNests Notifications")

	fmt.Println("\n=== Outlook (full mailbox) ===")
	outlookService := &EmailService{client: OutlookClient{accessToken: "outlook_token"}}
	outlookService.SendWelcomeEmail("rajin@outlook.com")

	fmt.Println("\n=== SendGrid (send-only — fits perfectly) ===")
	sendGridService := &SenderService{client: SendGridClient{apiKey: "sg_key_abc123"}}
	sendGridService.SendWelcomeEmail("rajin@technonext.com")
}
```

### Why This Is Better

In the inheritance version:

```
All services were forced to implement all behaviors
```

which created:

- panic methods
- rigid hierarchy
- tight coupling

With composition:

```
Each service selects only the capabilities it actually needs
```

No unnecessary methods.

No fake implementations.

This is why composition is usually more flexible than inheritance.

So this is also why go developer follows this approach. But this approach has some limitation.

### More Boilerplate

Composition can create many small structs.

Example: EmailSender, InboxReader, MailBoxManager

Then services combine them:

In large systems this may feel verbose.

### Method Conflicts

Two composed structs may contain methods with the same name.

Example:

```go
package main

import "fmt"

// ---------------- A ----------------
type A struct{}

func (A) Start() {
	fmt.Println("A started")
}

// ---------------- B ----------------
type B struct{}

func (B) Start() {
	fmt.Println("B started")
}

// ---------------- Composition ----------------
type App struct {
	A
	B
}

func main() {
	app := App{}

	// ❌ ERROR:
	// ambiguous selector app.Start()
	// app.Start();

	// Why?
	// Because both Logger and Server
	// have a method named Start()

	// So Go does not know which one to call.

	// ✅ Explicit calls:
	app.A.Start()
	app.B.Start()
}
```

then:

Now `app.Start()` becomes ambiguous.

You must resolve conflicts manually.

### Go prefers:

# "Compose behaviors"

instead of:

# "Extend hierarchies"

Because real-world systems are usually combinations of capabilities, not strict inheritance trees.