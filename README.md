# OWASP A05 SSTI Lab

A small educational Node.js lab demonstrating **Server-Side Template Injection (SSTI)** using **Express** and **EJS**.

This project intentionally shows how unsafe template construction can cause user input to be interpreted as template code instead of normal text. OWASP describes SSTI as a vulnerability that occurs when user input is embedded into a template in an unsafe way, and PortSwigger notes that this can allow attackers to inject template directives and potentially achieve code execution depending on the template engine.[web:44][web:46]

## Overview

This lab contains:

- A **vulnerable preview route** that builds a new EJS template string using user-controlled input.
- A **safe preview route** that treats user input as data and renders it through a normal trusted EJS view.
- A simple greeting-card style interface to demonstrate the issue clearly.

The goal is to understand:

- How SSTI happens.
- How to detect it with a basic payload.
- Why unsafe template construction is dangerous.
- How safer rendering patterns reduce risk.[web:44][web:46]

## Tech Stack

- Node.js
- Express
- EJS

Express supports server-side rendering with EJS through `app.set('view engine', 'ejs')` and `res.render(...)`, which makes it easy to demonstrate both secure and insecure patterns in a small local lab.[web:55][web:60]

## Vulnerability Explained

The vulnerable route takes user-controlled fields such as `name` and `message`, inserts them directly into a newly created template string, and then passes that string into `ejs.render(...)`.

### Vulnerable idea

```js
const vulnerableTemplate = `
  <h1>Greeting for ${name}</h1>
  <p>${message}</p>
`;

const html = ejs.render(vulnerableTemplate, {});
```

This is dangerous because the application is not only displaying user input — it is **constructing template code from user input** and then asking the template engine to interpret it. OWASP’s testing guide explains that SSTI occurs when user input is embedded unsafely into a template, while PortSwigger describes the issue as unsafe embedding of user input into server-side templates that allows template directives to be injected.[web:44][web:46]

## Proof of Concept

A simple SSTI detection payload for this lab is:

```text
<%= 7*7%>
```

If the vulnerable route is working as intended, the application renders:

```text
49
```

instead of showing the payload as plain text. PortSwigger documents arithmetic-style payloads such as `7*7` as a standard way to detect SSTI because successful evaluation proves the server is processing template syntax.[web:41][web:46]

### Example test

1. Open the app.
2. Enter a normal name.
3. Put `<%= 7*7 %>` in the message field.
4. Submit the vulnerable preview form.
5. Observe that the rendered card displays `49`.

You can also test malformed template input such as:

```text
<% invalid js here %>
```

OWASP notes that error behavior can help identify SSTI and the underlying template engine during testing.[web:44]

## Safe Comparison Route

The lab also includes a safer route that renders a trusted EJS template and passes user input as ordinary data:

```js
res.render('safe-preview', { name, message, style });
```

In the EJS template, values are rendered as variables rather than being used to generate a fresh attacker-controlled template string. This is a much safer pattern because the template structure remains trusted and static.[web:55][web:60]

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/rohitoff799-netizen/owasp-a05-ssti-lab.git
cd owasp-a05-ssti-lab
```

### 2. Install dependencies

```bash
npm install
```

If PowerShell blocks `npm`, use:

```powershell
npm.cmd install
```

### 3. Start the lab

```bash
node app.js
```

Or:

```bash
npm start
```

If you add a `start` script to `package.json`.

### 4. Open in browser

```text
http://127.0.0.1:3000
```

Express resolves EJS views from the `views` directory and uses `res.render()` to send rendered HTML to the browser.[web:55][web:60]

## Project Structure

```text
owasp-a05-ssti-lab/
├── app.js
├── package.json
├── public/
└── views/
    ├── index.ejs
    ├── preview.ejs
    └── safe-preview.ejs
```

## How to Reproduce

1. Start the application.
2. Visit `http://127.0.0.1:3000`.
3. Submit a normal greeting message to confirm the app works.
4. Submit the payload `<%= 7*7 %>` in the message field.
5. Observe that the vulnerable preview evaluates the expression and shows `49`.
6. Open the safe preview route and compare the behavior.

This demonstrates that the vulnerable route is evaluating user-controlled template syntax on the server, which is the core SSTI issue described by OWASP and PortSwigger.[web:44][web:46]

## Impact

Depending on the template engine and application context, SSTI can range from information disclosure to arbitrary code execution on the server. PortSwigger notes that the severity varies by template engine, and in some cases attackers may be able to escape restrictions and take full control of the web server.[web:46][web:41]

## Mitigation

To reduce SSTI risk:

- Do **not** build templates dynamically from untrusted input.[web:44][web:46]
- Treat user input as **data**, not template code.[web:44]
- Render only trusted, static server-side templates.[web:55][web:60]
- Avoid passing attacker-controlled content into rendering functions such as `ejs.render()` as part of a newly constructed template string.[web:46]
- Validate and constrain user input where appropriate.[web:44]

### Safer pattern

Instead of this:

```js
const html = ejs.render(`<p>${message}</p>`, {});
```

prefer this:

```js
res.render('safe-preview', { message });
```

This keeps the template trusted and only injects user data into predefined placeholders.[web:55][web:60]

## Learning Outcome

By building this lab, I practiced:

- Setting up an Express + EJS app.
- Identifying unsafe server-side template construction.
- Testing SSTI with a simple proof-of-concept payload.
- Comparing vulnerable and safer rendering approaches.
- Documenting a real OWASP-style injection issue in a local demo.[web:44][web:59]

## References

- OWASP Web Security Testing Guide – Server Side Template Injection.[web:44]
- PortSwigger Research – Server-Side Template Injection.[web:41]
- PortSwigger Issue Definition – Server-side template injection.[web:46]
- EJS/Express rendering guides.[web:55][web:60]

## Disclaimer

This project is for **educational and local testing purposes only**. It is intentionally vulnerable so that SSTI behavior can be studied safely in a controlled environment.[web:44][web:46]
