---
title: "Security Coding in Go. Input validation"
date: 2023-08-01T15:59:02+01:00
draft: false
tags: [appsec, go, golang, security-coding, validation]
description: "Input validation is one of most important technique in secure coding. Deep dive into it for Go language."
---

Input validation is one of most important technique in secure coding. Deep dive into it for Go language.

From [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html):

> Input validation is performed to ensure only properly formed data is entering the workflow in an information system.

> Input validation should happen as early as possible in the data flow, preferably as soon as the data is received from the external party.

> Data from all **potentially untrusted sources** should be subject to input validation, including not only Internet-facing web clients but also backend feeds over extranets, from suppliers, partners, vendors or regulators, each of which may be compromised on their own and start sending malformed data.

## Input Processing Model - Developer vs Hacker

{{< figure src="https://github.com/xvnpw/xvnpw.github.io/assets/17719543/dce28b3f-952b-4d0e-a396-3cae02413f70" class="image-center" >}}

In this article I will focus on finding loophols in typical input validation techniques. Hackers are targeting those to bypass filtering and achieve goals. Typical example of such problem is `.` vs `\.` in regex. We will take a look on that later.

## Why we should use it?

- for data consistency 
- make life harder for hackers (and more fun as they will search for bypass)
- **essential** for defending against some attack types - e.g. domain validation in case of Server Side Request Forger (SSRF)

{{< figure src="https://github.com/xvnpw/xvnpw.github.io/assets/17719543/8226da79-2e3d-49cb-a8f7-b114563186fc" class="image-center" >}}

## Common validation issues

### Regex dot

In regex, it easy to forget about `.` meaning, especially in domain validation: 

- `.` - any single character
- `\.` - dot character

❌ **Bad code**
```go
matched, _ := regexp.Match(`^(test|stage|prod).example.com$`, []byte(domain))
// prod-example.com
```

✅ **Correct code**
```go
matched, _ := regexp.Match(`^(test|stage|prod)\.example\.com$`, []byte(domain))
```

### Regex matching whole input

In regex, in `go`, your regular expression is not matched against whole input. You need to add `^` to beginning and `$` to ending.

❌ **Bad code**
```go
re := regexp.MustCompile(`test\.example\.com`)
// test.example.com.my-bad-domain.com
```

✅ **Correct code**
```go
re := regexp.MustCompile(`^test\.example\.com$`)
```

### Wrong regex - range problem

`[A-z]`: A-z matches a single character in the range between A (index 65) and z (index 122) - which includes, e.g. `[`, `\`, `]`

❌ **Bad code**
```
^[A-z0-9]{4}$
```

✅ **Correct code**
```
^[a-zA-Z0-9]{4}$
```

⭐ **Important:** Validation is not proven to work if not tested

### Email Address Validation

Email format is defined by [RFC 5321](https://tools.ietf.org/html/rfc5321#section-4.1.2), but it's quite wide. Those all are **valid**:

- `"><script>alert(1);</script>"@example.org`
- `user+subaddress@example.org`
- `user@[IPv6:2001:db8::1]`
- `" "@example.org`

Try to register `"><script>alert(1);</script>"` in gmail. What will you get?

Following **Bad code** is wrong, because it only relay on RFC 5321. OWASP is giving more [strict rules](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html#email-address-validation) for validation.

❌ **Bad code**
```go
import "net/mail"

func main() {
	input := `"><script>alert(1);</script>"@example.org`

	e, err := mail.ParseAddress(input)
}
// "><script>alert(1);</script>"@example.org
```

⭐ **Important:** There is no perfect validation of email address

✅ **Correct code** - based on OWASP Validation Regex Repository
```go
func isEmailValid(input string) bool {
	// RFC 5321 validation
	_, err := mail.ParseAddress(input)
	if err != nil {
		return false
	}

	r := regexp.MustCompile(`^[a-zA-Z0-9_+&*-]+(?:\.[a-zA-Z0-9_+&*-]+)*@(?:[a-zA-Z0-9-]+\.)+[a-zA-Z]{2,}$`)
	if !r.Match([]byte(input)) {
		return false
	}

	return true
}
```

✅ **Correct code** - based on OWASP Cheat Sheet
```go
func isEmailValid(input string) bool {
	// RFC 5321 validation
	_, err := mail.ParseAddress(input)
	if err != nil {
		return false
	}

	parts := strings.Split(input, "@")

	// we don't allow some special characters in email part
	if strings.ContainsAny(parts[0], `"'/\;<>`) {
		return false
	}

	// something is not right with utf8
	if len(input) != utf8.RuneCountInString(input) {
		return false
	}

	// domain part should contain only letters, numbers, hyphens and periods
	r := regexp.MustCompile(`^[a-zA-Z0-9\.-]*$`)
	if !r.Match([]byte(parts[1])) {
		return false
	}

	// email part should be no more than 63 characters
	if len(parts[0]) > 63 {
		return false
	}

	// total length should be no more than 254 characters
	if len(input) > 254 {
		return false
	}

	return true
}
```

❓ If you don't like above. Can you understand this regex?
```go
// https://github.com/asaskevich/govalidator
Email string = "^(((([a-zA-Z]|\\d|[!#\\$%&'\\*\\+\\-\\/=\\?\\^_`{\\|}~]|[\\x{00A0}-\\x{D7FF}\\x{F900}-\\x{FDCF}\\x{FDF0}-\\x{FFEF}])+(\\.([a-zA-Z]|\\d|[!#\\$%&'\\*\\+\\-\\/=\\?\\^_`{\\|}~]|[\\x{00A0}-\\x{D7FF}\\x{F900}-\\x{FDCF}\\x{FDF0}-\\x{FFEF}])+)*)|((\\x22)((((\\x20|\\x09)*(\\x0d\\x0a))?(\\x20|\\x09)+)?(([\\x01-\\x08\\x0b\\x0c\\x0e-\\x1f\\x7f]|\\x21|[\\x23-\\x5b]|[\\x5d-\\x7e]|[\\x{00A0}-\\x{D7FF}\\x{F900}-\\x{FDCF}\\x{FDF0}-\\x{FFEF}])|(\\([\\x01-\\x09\\x0b\\x0c\\x0d-\\x7f]|[\\x{00A0}-\\x{D7FF}\\x{F900}-\\x{FDCF}\\x{FDF0}-\\x{FFEF}]))))*(((\\x20|\\x09)*(\\x0d\\x0a))?(\\x20|\\x09)+)?(\\x22)))@((([a-zA-Z]|\\d|[\\x{00A0}-\\x{D7FF}\\x{F900}-\\x{FDCF}\\x{FDF0}-\\x{FFEF}])|(([a-zA-Z]|\\d|[\\x{00A0}-\\x{D7FF}\\x{F900}-\\x{FDCF}\\x{FDF0}-\\x{FFEF}])([a-zA-Z]|\\d|-|\\.|_|~|[\\x{00A0}-\\x{D7FF}\\x{F900}-\\x{FDCF}\\x{FDF0}-\\x{FFEF}])*([a-zA-Z]|\\d|[\\x{00A0}-\\x{D7FF}\\x{F900}-\\x{FDCF}\\x{FDF0}-\\x{FFEF}])))\\.)+(([a-zA-Z]|[\\x{00A0}-\\x{D7FF}\\x{F900}-\\x{FDCF}\\x{FDF0}-\\x{FFEF}])|(([a-zA-Z]|[\\x{00A0}-\\x{D7FF}\\x{F900}-\\x{FDCF}\\x{FDF0}-\\x{FFEF}])([a-zA-Z]|\\d|-|_|~|[\\x{00A0}-\\x{D7FF}\\x{F900}-\\x{FDCF}\\x{FDF0}-\\x{FFEF}])*([a-zA-Z]|[\\x{00A0}-\\x{D7FF}\\x{F900}-\\x{FDCF}\\x{FDF0}-\\x{FFEF}])))\\.?$"
```

#### Online validation

❌ There are libraries like https://github.com/badoux/checkmail that can do online validation, asking SMTP server if mailbox exists on it. It's not perfect and can be misleading. 

#### Semantic validation

Semantic validation provides basic level of assurance that:
- email address is correct
- application can successfully send emails to it
- user has access to the mailbox

✅ The most common way to do this is to send an email to the user, and require that they click a link in the email, or enter a code that has been sent to them.

### Unicode Case Mapping Collisions 💥

Unicode Case Mapping Collisions occur when two different characters are uppercased or lowercased into the same character.

```go
strings.ToUpper(ı) // ı is dotless i, used in Turkish alphabet 
// I
strings.ToUpper(ſ) // ſ is latin small letter Long S
// S
```

`go` function list, that can end up with collision:
- `strings.ToLower(input)`
- `strings.ToUpper(input)`
- `bytes.ToLower(input)`
- `bytes.ToUpper(input)`
- `norm.NFC.String(input)`
- `norm.NFKC.String(input)`

❌ **Bad code**
```go
if strings.ToUpper(s1) == strings.ToUpper(s2)
// ıtest == itest
```

✅ **Correct code**
```go
if s1 == s2
```

How to prevent it:
- [accept fact](https://www.compart.com/en/unicode/U+0131) that some letters have collision on uppercased or lowercased: `ı` -> `I`
- be careful in any sensitive context - e.g. `username` - best to handle it as is, without any transformation
- in case you have requirement on case insensitive - be consistent - normalize or transform you input always in same way
- use `utf8` package, e.g. `len(x)` vs `utf8.RuneCountInString(x)`

#### Read more

- https://dev.to/jagracey/hacking-github-s-auth-with-unicode-s-turkish-dotless-i-460n
- https://www.gosecure.net/blog/2020/08/04/unicode-for-security-professionals/
- https://henvic.dev/posts/go-utf8/

### Blacklisting vs whitelisting

Prefer whitelist over blacklist. 

## Model validation using `go-swagger`

`go-swagger` package can parse swagger file and generate model. This model includes validations.

⭐ **Important:** It doesn't generate validation code for path and query parameters.

## Interesting regex

```go
// https://github.com/asaskevich/govalidator
UUID4 string = "^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$"
userRegexp = regexp.MustCompile("^[a-zA-Z0-9!#$%&'*+/=?^_`{|}~.-]+$")
```

## Validation Techniques Theory  

* Whitelisting - whenever possible validate the input against a whitelist of allowed characters.
* Boundary checking - both data and numbers length should be verified.
* Character escaping - for special characters such as standalone quotation marks.
* Numeric validation - if input is numeric.
* Check for Null Bytes - `(%00)`
* Checks forpath alteration characters - `../` or `\\..`
* Checks for Extended UTF-8 - check for alternative representations of special characters

## Validation in `go`

* https://github.com/go-playground/validator
* https://github.com/gorilla/schema
* https://github.com/go-playground/form
* https://github.com/go-ozzo/ozzo-validation
* https://github.com/asaskevich/govalidator
* package `strconv`, e.g. `Atoi`, `ParseBool`, `ParseFloat`, `ParseInt`
* package `strings`, e.g. `Contains`, `HasPrefix`, `HasSuffix`
* package `regexp`
* package `utf8`, e.g. `Valid`, `ValidRune`, `ValidString`

If you have any comments or feedback, you are welcome to write to me on [Twitter](https://twitter.com/xvnpw).
