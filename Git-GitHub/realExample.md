
# 🚀 PART 1 — Real Open Source Contribution Example

Let’s assume you want to contribute to a Spring Boot project.

Example repository:

`awesome-springboot-project`

---

## 🔹 Step 1: Find an Issue

Go to Issues tab → filter:

* `good first issue`
* `bug`
* `enhancement`

Comment:

> Hi, I would like to work on this issue. Please assign it to me.

Wait for confirmation.

---

## 🔹 Step 2: Fork & Clone

```bash
git clone https://github.com/your-username/awesome-springboot-project.git
cd awesome-springboot-project
```

Add upstream:

```bash
git remote add upstream https://github.com/original-owner/awesome-springboot-project.git
```

---

## 🔹 Step 3: Create Branch

```bash
git checkout -b fix-null-check-in-user-service
```

---

## 🔹 Step 4: Make Changes

Suppose issue says:

“NullPointerException when email is null”

You:

* Add null validation
* Add test case
* Ensure app runs

---

## 🔹 Step 5: Commit Professionally

```bash
git add .
git commit -m "fix: prevent NullPointerException when user email is null"
```

Why this is good:

* Starts with type (`fix`)
* Short description
* Clear problem

---

## 🔹 Step 6: Push & PR

```bash
git push origin fix-null-check-in-user-service
```

Create PR with:

* Problem description
* Solution explanation
* Testing steps

---

## 🔹 Step 7: Code Review Phase

Maintainer says:

> Please rename variable and add test coverage.

You fix:

```bash
git add .
git commit -m "refactor: improve variable naming and add unit test"
git push
```

PR updates automatically.

---

# 📌 PART 2 — Open Source Cheat Sheet (Save This)

---

## 🔥 Golden Rules

✔️ One issue → One branch
✔️ Small PRs get accepted faster
✔️ Always sync with upstream
✔️ Never push directly to main
✔️ Follow coding style

---

## 🔥 Commit Types

| Type     | Use For       |
| -------- | ------------- |
| fix      | Bug fix       |
| feat     | New feature   |
| docs     | Documentation |
| refactor | Improve code  |
| test     | Add tests     |
| chore    | Minor updates |

---
