---
title: "SSH Configuration File"
date: 2025-02-12 10:00:00 +0800
categories: [configuration, ssh]
tags: [configuration, ssh]
---

In the SSH configuration file, you can customize the connection settings for specific hosts, including identity authentication keys, connection options, and ports. The SSH configuration file is usually located in `~/.ssh/config`, and if the file does not exist, you can create it manually. Here are some common configuration options and examples.

### 1. **Edit or create the SSH configuration file**
First, ensure you have a configuration file. If the file does not exist, you can create it using the following command:

```bash
touch ~/.ssh/config
```

Then, you can open it using a text editor, for example:

```bash
nano ~/.ssh/config
```

### 2. **Common items in the configuration file**

#### 1) **Set up SSH configuration for GitHub**
If you have multiple GitHub accounts or need to customize the configuration, you can set up specific settings for `github.com`:

```bash
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa
  IdentitiesOnly yes
  PreferredAuthentications publickey
```

- `Host github.com`：Configure `github.com` as the host (you can use any name, and reference it later).
- `HostName github.com`：The actual host address.
- `User git`：The user for GitHub login is `git`, not your GitHub username.
- `IdentityFile ~/.ssh/id_rsa`：Specify the path to the private key file used.
- `IdentitiesOnly yes`：Ensure the SSH client only uses the key specified in `IdentityFile`, not other keys.
- `PreferredAuthentications publickey`：Set the preferred authentication method to public key.

#### 2) **Customize the SSH connection port**
If you need to connect to GitHub or other servers through a non-default port, you can specify the port, for example, using port `443` (the alternative port for GitHub):

```bash
Host github.com
  HostName github.com
  User git
  Port 443
  IdentityFile ~/.ssh/id_rsa
```

#### 3) **Multiple GitHub account configuration**
If you have multiple GitHub accounts and want to use different SSH keys, you can configure different Host names for each account. For example:

```bash
# 账号 1
Host github-acc1
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_acc1

# 账号 2
Host github-acc2
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_acc2
```

Then, use the following commands to specify different GitHub accounts:

```bash
git clone git@github-acc1:username/repository.git
git clone git@github-acc2:username/repository.git
```

#### 4) **Specify connection timeout**
If you want to set a connection timeout, you can use `ConnectTimeout`:

```bash
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa
  ConnectTimeout 10
```

#### 5) **Use a proxy server**
If you need to connect to SSH through a proxy server, you can set `ProxyCommand`:

```bash
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa
  ProxyCommand nc -X 5 -x proxy.example.com:1080 %h %p
```

- `-X 5`：Specify the use of SOCKS5 proxy.
- `-x proxy.example.com:1080`：The address and port of the proxy server.

### 3. **Configuration file format description**
- `Host`：Specify the host alias for the configuration item, you can use different aliases as needed. For example, you can use `github.com` or `github-acc1` instead of the specific host address.
- `HostName`：Specify the actual host address, for example, `github.com`.
- `User`：The username used during login (for GitHub, it is `git`).
- `IdentityFile`：Specify the path to the private key file.
- `Port`：Specify the SSH connection port.
- `ConnectTimeout`：Set the number of seconds for the connection timeout.
- `ProxyCommand`：Set up SSH connection through a proxy server.

### 4. **Save and test the configuration**
After completing the configuration, save the file and exit the editor (for example, press `Ctrl+X` in `nano` and then press `Y` to confirm the save).

Next, you can test if the configuration is effective:

```bash
ssh -T git@github.com
```

If the configuration is correct, SSH will use the options in the configuration file to connect, and should be able to successfully authenticate.

### 5. **Summary**
By modifying the `~/.ssh/config` file, you can easily create different configuration items for multiple hosts, different keys, and connection settings, which can simplify commands and improve efficiency.
