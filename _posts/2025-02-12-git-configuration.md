---
title: "Git Configuration"
date: 2025-02-12 10:00:00 +0800
categories: [configuration, git]
tags: [configuration, git]
---

In GitHub, configuring SSH is primarily done to securely access and operate GitHub repositories using SSH keys. Here are the steps to configure GitHub SSH:

### 1. Check if you already have SSH keys
First, check if you already have SSH keys. If you are unsure, you can use the following command to view if you have existing keys:

```bash
ls -al ~/.ssh
```

Check if there is a file named `id_rsa` or `id_ed25519`, if not, generate a new key.

### 2. Generate a new SSH key
If you don't have existing SSH keys, you can use the following command to generate a new key:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

- `-t rsa` means using RSA algorithm.
- `-b 4096` means the key length (4096 is a secure choice).
- `-C "your_email@example.com"` is the comment for the key, usually use your GitHub email.

When prompted, press Enter to accept the default location (usually `~/.ssh/id_rsa`). If you want to set a passphrase, you can enter it when prompted, or press Enter to skip it.

### 3. Add the SSH key to the SSH agent
To avoid entering a password each time you perform an operation, you can add the SSH key to the SSH agent:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

### 4. Add the SSH public key to GitHub
Next, add the public key to GitHub:

1. Copy the content of the public key (usually the content of `~/.ssh/id_rsa.pub` or `~/.ssh/id_ed25519.pub` file). You can use the following command to view and copy:

   ```bash
   cat ~/.ssh/id_rsa.pub
   ```

2. Login to your GitHub account, enter **Settings** -> **SSH and GPG keys** -> **New SSH key**.
3. Fill in a recognizable name (e.g., "My Mac") in the "Title" field, paste the public key content you copied earlier, and click **Add SSH key**.

### 5. Test SSH connection
Ensure the SSH configuration is correct, you can use the following command to test the connection:

```bash
ssh -T git@github.com
```

If it is the first time connecting, the system will prompt you whether to trust the GitHub server, input `yes`即可。If everything is normal, you will see something like:

```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

### 6. Configure Git to use SSH
Ensure Git uses SSH for repository operations. Check if the remote URL of the repository is an SSH address instead of HTTPS:

```bash
git remote -v
```

If it is a HTTPS address, you can modify it to an SSH address:

```bash
git remote set-url origin git@github.com:username/repository.git
```




