# Termux Usage Logs

## 1.1 Contact with Android system files

Since Android 11 + seperate the system files from the other softwares.They do not have the permission to interact with the system files.So we can execute the commonds below to create a `storage` directory-link which links to the system file's storage.

```bash
termux-setup-storage
```

So that we can use Obsidian and git to manage files easily.

---

## 2. skills added

### 2.1 convert markdown to pdf

The tool used in vscode is on the basis of wkhtmltopdf.But openclaw runs in the wsl2 environment which does not have the tool and you should install it first.

```bash
sudo apt upgrade
sudo apt update
sudo apt install wkhtmltopdf
```