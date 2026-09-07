# GitHub SSH Key Setup (macOS)

> GitHub username change করার পর পুরনো SSH key remove করে নতুন key setup করার সম্পূর্ণ গাইড।

---

## Step 1: GitHub থেকে পুরনো key delete করুন

**GitHub.com → Settings → SSH and GPG keys** → প্রতিটা পুরনো key-এর পাশে **Delete** → confirm করুন।

> Password বা 2FA চাইতে পারে।

## Step 2: PC থেকে পুরনো key file delete করুন

প্রথমে দেখুন কী কী key আছে:

```bash
ls ~/.ssh
```

Key file গুলো delete করুন (আপনার key-এর নাম অনুযায়ী):

```bash
rm ~/.ssh/id_ed25519 ~/.ssh/id_ed25519.pub
```

## Step 3: SSH agent থেকে clear করুন

```bash
ssh-add -D
```

## Step 4: Config file check করুন

```bash
cat ~/.ssh/config
```

এখানে যদি শুধু GitHub-এর পুরনো key-গুলোর entry থাকে, পুরো file-টাই মুছে দিতে পারেন:

```bash
rm ~/.ssh/config
```

> ⚠️ অন্য কোনো server-এর configuration থাকলে পুরো file না মুছে শুধু GitHub-এর অংশটুকু edit করে বাদ দিন।

সব clean হয়েছে কিনা যাচাই করুন:

```bash
ls ~/.ssh
```

এখন শুধু `known_hosts` আর `known_hosts.old` থাকার কথা — এ দুটো রেখে দিলে সমস্যা নেই।

## Step 5: নতুন SSH key generate করুন

```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
```

- Email-এর জায়গায় আপনার **GitHub account-এর email** দিন
- File location জিজ্ঞেস করলে **Enter** চাপুন (default location-ই থাকুক)
- Passphrase চাইলে দিন, বা খালি রেখে **Enter** চাপুন

## Step 6: SSH agent-এ key add করুন

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

## Step 7: Public key copy করুন

```bash
cat ~/.ssh/id_ed25519.pub
```

macOS-এ সরাসরি clipboard-এ copy করতে:

```bash
pbcopy < ~/.ssh/id_ed25519.pub
```

> ⚠️ শুধু `.pub` file-টাই (public key) copy করবেন। `id_ed25519` (private key) **কখনোই** কোথাও paste করবেন না।

## Step 8: GitHub-এ key add করুন

**GitHub.com → Settings → SSH and GPG keys → New SSH key**

- **Title:** device চেনার মতো একটা নাম (যেমন `My MacBook`)
- **Key type:** `Authentication Key` (default)
- **Key:** copy করা public key paste করুন
- **Add SSH key** button চাপুন

## Step 9: Connection test করুন

```bash
ssh -T git@github.com
```

প্রথমবার `Are you sure you want to continue connecting?` জিজ্ঞেস করলে `yes` লিখে Enter দিন। তারপর দেখবেন:

```
Hi NEW-USERNAME! You've successfully authenticated, but GitHub does not provide shell access.
```

এটা দেখলেই বুঝবেন সব ঠিকঠাক ✅

## Step 10: Repo-গুলোর remote URL update করুন

প্রতিটা local repo-তে গিয়ে দেখুন এখন কী set করা আছে:

```bash
cd /path/to/your/repo
git remote -v
```

নতুন username দিয়ে update করুন:

```bash
git remote set-url origin git@github.com:NEW-USERNAME/repo-name.git
```

যাচাই করতে:

```bash
git remote -v
git pull
```

`git pull` ঠিকঠাক কাজ করলেই সব সেট 🎉

## Step 11: Git config-এ username ও email ঠিক করুন

Commit-এ যে নাম দেখায় সেটা আসে PC-র local git config থেকে। Email GitHub account-এর সাথে না মিললে commit-এর পাশে profile picture/link দেখাবে না।

আগে দেখুন PC-তে কী set করা আছে:

```bash
git config --global user.name
git config --global user.email
```

তারপর ঠিক করুন:

```bash
git config --global user.name "arifucoder"
git config --global user.email "arifucoder@gmail.com"
```

> ⚠️ `user.email`-টা অবশ্যই GitHub account-এ verified email হতে হবে (**Settings → Emails**)। আর এই setting device-ভিত্তিক — প্রতিটা PC/Mac-এ আলাদা করে set করতে হবে।

এই পরিবর্তন শুধু নতুন commit-এ কাজ করবে; পুরনো commit-এর নাম বদলাবে না।

----
----

Private-GitHub-Repo-SSH-Setup-Guide
---
# Private GitHub Repository → VPS/AWS Instance এ SSH দিয়ে Clone করা (CI/CD Setup)

যদি একটি **private GitHub repository** থাকে এবং সেটাকে CI/CD এর মাধ্যমে বা সরাসরি কোনো **VPS/AWS instance** এ clone করতে চান, তাহলে সেই মেশিনে একটি **SSH key pair (public + private key)** তৈরি করে GitHub এর সাথে যুক্ত (link) করে নিতে হবে। নিচে ধাপে ধাপে পুরো প্রক্রিয়াটি দেওয়া হলো।

---

## ধাপ ১: VPS এর Home Directory তে যাওয়া

প্রথমে আপনার VPS/AWS instance এ SSH করে ঢুকুন এবং home directory তে যান (`~` মানে হলো home directory):

```bash
cd ~
```

এরপর `ll` (বা `ls -la`) কমান্ড দিয়ে দেখুন `.ssh` নামে একটি folder আছে কিনা:

```bash
ll
```

---

## ধাপ ২: `.ssh` Folder এ ঢোকা

```bash
cd .ssh
ll
```

এখানে ঢুকে দেখুন ভেতরে আগে থেকে কোনো key file আছে কিনা।

---

## ধাপ ৩: নতুন SSH Key Generate করা

নিচের কমান্ড দিয়ে নতুন একটি SSH key তৈরি করুন:

```bash
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
```

কমান্ড রান করার পর কয়েকটি প্রশ্ন আসবে:

1. **"Enter file in which to save the key"** → এখানে একটা নাম দিন (যেমন: `github_key`)
2. **Passphrase** → খালি রেখে সরাসরি Enter চাপুন
3. আবার Enter চাপুন (confirm করার জন্য)

এই ধাপ শেষ হলে **২টি file** তৈরি হবে:
- `github_key` → এটা **private key** (কোথাও শেয়ার করবেন না)
- `github_key.pub` → এটা **public key** (এটাই GitHub এ যুক্ত করতে হবে)

---

## ধাপ ৪: Public Key কপি করা

`.pub` extension এর file টি খুলে ভেতরের কন্টেন্ট কপি করুন:

```bash
cat github_key.pub
```

পুরো output টি (ssh-rsa দিয়ে শুরু হওয়া লাইনটি) কপি করে রাখুন।

---

## ধাপ ৫: GitHub এ Public Key যুক্ত করা

1. GitHub প্রোফাইলে যান → **Settings**
2. বাম পাশের মেনু থেকে **SSH and GPG keys** এ ক্লিক করুন
3. **New SSH key** বাটনে ক্লিক করুন
4. একটি **Title/Name** দিন (যেমন: `My VPS Key`)
5. **Key** ফিল্ডে আগে কপি করা public key টি পেস্ট করুন
6. **Add SSH key** বাটনে ক্লিক করে সেভ করুন

---

## ধাপ ৬: VPS এ ফিরে গিয়ে SSH Agent চালু করা

আবার VPS এর home directory তে ফিরে যান:

```bash
cd ~
```

SSH agent চালু (restart) করুন:

```bash
eval "$(ssh-agent -s)"
```

> লক্ষ্য করুন: `$` এবং `(` এর মাঝে কোনো space থাকবে না — সঠিক লেখা হলো `$(ssh-agent -s)`।

---

## ধাপ ৭: Private Key কে SSH Agent এ যুক্ত করা

```bash
ssh-add ~/.ssh/github_key
```

সফল হলে এরকম একটি message দেখাবে যে identity টি (key) agent এ যুক্ত হয়েছে।

---

## ধাপ ৮ (Optional কিন্তু সুপারিশকৃত): Connection Test করা

Clone করার আগে যাচাই করে নিন GitHub এর সাথে SSH connection ঠিকভাবে কাজ করছে কিনা:

```bash
ssh -T git@github.com
```

সফল হলে GitHub থেকে একটি welcome message আসবে, যেখানে আপনার username উল্লেখ থাকবে।

---

## ধাপ ৯: Repository Clone করা

এখন SSH link ব্যবহার করে private repository টি clone করতে পারবেন:

```bash
git clone git@github.com:username/repository-name.git
```

এভাবে CI/CD pipeline এ বা সরাসরি VPS/AWS instance এ private repository টি ব্যবহার করা যাবে।

---

## সংক্ষিপ্ত সারমর্ম (Summary)

| ধাপ | কাজ |
|---|---|
| ১-২ | `.ssh` folder এ যাওয়া |
| ৩ | `ssh-keygen` দিয়ে key pair তৈরি করা |
| ৪-৫ | Public key কপি করে GitHub এ যুক্ত করা |
| ৬-৭ | SSH agent চালু করে private key যুক্ত করা |
| ৮ | Connection টেস্ট করা |
| ৯ | `git clone` দিয়ে repository নামানো |

