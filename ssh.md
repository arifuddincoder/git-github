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