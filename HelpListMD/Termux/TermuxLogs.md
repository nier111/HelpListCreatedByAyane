# Termux Usage Logs

## 1.1 Contact with Android system files

Since Android 11 + seperate the system files from the other softwares.They do not have the permission to interact with the system files.So we can execute the commonds below to create a `storage` directory-link which links to the system file's storage.

```bash
termux-setup-storage
```

So that we can use Obsidian and git to manage files easily.
