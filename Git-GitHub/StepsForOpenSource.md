
# 🚀 PART 1 — Important Steps for Open Source Contribution

---

## ✅ 1️⃣ Choose the Right Repository

* Pick project related to your stack (Spring Boot, Java etc.)
* Check:

  * Active commits
  * Recent issues
  * Clear README
  * Contribution guidelines

Look for labels like:

* `good first issue`
* `help wanted`
* `beginner friendly`

---

## ✅ 2️⃣ Understand the Project

Before writing code:

* Read README.md
* Read CONTRIBUTING.md
* Run project locally
* Understand folder structure
* Understand coding style

Real dev rule:

> Never open PR without understanding architecture.

---

## ✅ 3️⃣ Fork the Repository

Click **Fork** button.

This creates:

```id="x83v2q"
your-github/repository-name
```

You never push directly to original repo.

---

## ✅ 4️⃣ Clone Your Fork

```bash
git clone https://github.com/your-username/repository-name.git
```

Move into project:

```bash
cd repository-name
```

---

## ✅ 5️⃣ Add Upstream Remote (VERY IMPORTANT)

```bash
git remote add upstream https://github.com/original-owner/repository-name.git
```

Check remotes:

```bash
git remote -v
```

You should see:

* origin → your fork
* upstream → original repo

This keeps your fork updated.

---

## ✅ 6️⃣ Create a New Branch (Never Work on main)

```bash
git checkout -b fix-issue-101
```

Branch naming examples:

* `fix-login-bug`
* `add-user-validation`
* `docs-update-readme`

Professional rule:

> One issue = One branch

---

## ✅ 7️⃣ Make Changes

* Follow project code style
* Write clean code
* Add comments if needed
* Add test cases (if project has tests)

---

## ✅ 8️⃣ Commit Properly (Very Important)

Professional commit format:

```bash
git add .
git commit -m "fix: resolve null pointer in UserService"
```

### Good Commit Types:

* `fix:` → bug fix
* `feat:` → new feature
* `docs:` → documentation
* `refactor:` → code improvement
* `test:` → adding tests
* `chore:` → small maintenance

❌ Bad commit:

```
updated file
```

✅ Good commit:

```
fix: prevent crash when user email is null
```

---

## ✅ 9️⃣ Push Branch to Your Fork

```bash
git push origin fix-issue-101
```

---

## ✅ 🔟 Create Pull Request (PR)

Go to your fork on GitHub.

Click:
👉 Compare & Pull Request

Write:

* What problem
* What solution
* How tested
* Screenshots if UI change

---

## ✅ 1️⃣1️⃣ Respond to Review

Maintainers may:

* Request changes
* Suggest improvements
* Ask questions

Never argue emotionally.
Be professional.

Make changes:

```bash
git add .
git commit -m "refactor: improve variable naming as suggested"
git push origin fix-issue-101
```

PR updates automatically.

---

## ✅ 1️⃣2️⃣ Keep Fork Updated

Before next contribution:

```bash
git checkout main
git fetch upstream
git merge upstream/main
git push origin main
```

---

# 📌 Professional Commit Flow Example

```bash
git fork
git clone
git remote add upstream
git checkout -b fix-issue-45
# make changes
git add .
git commit -m "fix: correct validation logic in PaymentService"
git push origin fix-issue-45
# create PR
```

---

# 🧠 Important Open Source Rules

✔️ Never push directly to main
✔️ Always create separate branch
✔️ Always sync with upstream
✔️ Always write meaningful commits
✔️ Keep PR small (1 issue per PR)
✔️ Be respectful in discussion

---
