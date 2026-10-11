# Koko — Laravel React Starter Kit

A Laravel application built with React, TypeScript, Inertia.js, Tailwind CSS, and PostgreSQL. The project uses Docker Sail for the development environment, with WSL2 providing the Linux environment on Windows.

## Tech Stack

- **Backend:** Laravel, PHP
- **Frontend:** React, TypeScript, Inertia.js
- **Styling:** Tailwind CSS
- **Build tool:** Vite Plus
- **Database:** PostgreSQL
- **Containerization:** Docker Desktop, Laravel Sail
- **Development environment:** WSL2, Ubuntu
- **Database client:** DataGrip
- **Version control:** Git, GitHub

## 1. How the Development Environment Works

This project uses Windows with WSL2 and Docker Desktop.

```text
Windows 11
├── VS Code
├── Web Browser
├── DataGrip
└── Docker Desktop
    └── WSL2 (Ubuntu)
        ├── Project files
        └── Docker Engine
            ├── Laravel / PHP
            ├── PostgreSQL
            └── Vite / Node.js
```

The project files are stored inside the Linux filesystem rather than the Windows filesystem. This improves file access performance and helps Vite detect changes more reliably.

**Project location:**

```text
/home/<linux-username>/projects/koko
```

Avoid storing the active project under `/mnt/c/laragon/www` when using this workflow, as accessing Windows-mounted files can reduce performance for Linux-based development tools.

## 2. Prerequisites

Install the following software:

1. Windows 11
2. WSL2 with Ubuntu
3. Docker Desktop with WSL2 integration enabled
4. Visual Studio Code
5. Visual Studio Code WSL extension
6. Git
7. DataGrip (optional, for database management)

Docker Desktop provides the Linux container environment. You do not need to install PHP, PostgreSQL, or Node.js directly in Windows to run the application through Sail.

## 3. Set Up WSL2

Open PowerShell as Administrator and install WSL if it is not already installed:

```powershell
wsl --install -d Ubuntu
```

Restart Windows if requested, then open Ubuntu and create your Linux username.

Update Ubuntu:

```bash
sudo apt update
sudo apt upgrade -y
```

Install Git and the utilities needed for cloning and working with the project:

```bash
sudo apt install -y git curl unzip
```

Configure Git:

```bash
git config --global user.name "Your Name"
git config --global user.email "YOUR_GITHUB_EMAIL"
```

## 4. Configure GitHub SSH

Check whether an SSH key already exists:

```bash
ls -la ~/.ssh
```

If you do not have an SSH key, generate one:

```bash
ssh-keygen -t ed25519 -C "YOUR_GITHUB_EMAIL"
```

Start the SSH agent and add the key:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

Display your public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Add the public key to your GitHub account under **Settings → SSH and GPG keys**.

Test the connection:

```bash
ssh -T git@github.com
```

Do not upload or share your private key (`id_ed25519`).

## 5. Clone the Repository

Create a directory for your projects inside Ubuntu:

```bash
mkdir -p ~/projects
cd ~/projects
```

Clone the repository:

```bash
git clone git@github.com:harrazmasri/koko.git
cd koko
```

Confirm that the project is inside the Linux filesystem:

```bash
pwd
```

Expected location:

```text
/home/<linux-username>/projects/koko
```

## 6. Open the Project in VS Code

From the Ubuntu terminal:

```bash
cd ~/projects/koko
code .
```

If the `code` command is unavailable, install Visual Studio Code on Windows and install Microsoft's WSL extension.

Confirm that VS Code is connected to Ubuntu. The bottom-left corner should show `WSL: Ubuntu`.

Always open this Linux project through the WSL connection instead of opening a separate copy under `C:\laragon\www`.

## 7. Start Docker Desktop

Open Docker Desktop on Windows.

Make sure:

- Docker Desktop is running.
- The WSL2 engine is enabled.
- Ubuntu is enabled under Docker Desktop's WSL integration settings.

Verify Docker from Ubuntu:

```bash
docker version
docker compose version
```

Both commands should return version information without connection errors.

## 8. Configure Environment Variables

Create `.env` if it does not exist:

```bash
cp .env.example .env
```

Edit `.env` and configure the database connection:

```dotenv
DB_CONNECTION=pgsql
DB_HOST=pgsql
DB_PORT=5432
DB_DATABASE=koko_db
DB_USERNAME=sail
DB_PASSWORD=password
```

The values must match the database service and credentials configured in `compose.yaml`.

Do not commit `.env` or other files containing secrets to GitHub.

## 9. Install Dependencies and Start Sail

The repository should contain the Laravel Sail executable under `vendor/bin/sail`. If it is missing, dependencies must be installed before using Sail.

If PHP and Composer are not installed directly in Ubuntu, use a compatible Composer environment or install the required PHP and Composer tools first. Once Sail is available, use Sail for the remaining PHP and Node.js commands.

Install PHP dependencies if needed:

```bash
./vendor/bin/sail composer install
```

Install frontend dependencies:

```bash
./vendor/bin/sail npm install
```

Build the Docker image:

```bash
./vendor/bin/sail build
```

Start the containers:

```bash
./vendor/bin/sail up -d
```

Generate the Laravel application key:

```bash
./vendor/bin/sail artisan key:generate
```

Clear cached configuration and run database migrations:

```bash
./vendor/bin/sail artisan config:clear
./vendor/bin/sail artisan migrate
```

Build frontend assets for a production-style build:

```bash
./vendor/bin/sail npm run build
```

Check the running containers:

```bash
./vendor/bin/sail ps
```

## 10. Run Vite for Development

For frontend development with hot reload, run:

```bash
./vendor/bin/sail npm run dev
```

Keep this terminal open while developing.

Vite watches the frontend files and updates the browser when supported changes are detected. You normally do not need to rebuild the Docker image when editing React or PHP files.

The `npm run start` command is not defined in this project's `package.json`; use `npm run dev` for development.

## 11. Access the Application

Open the local URL configured by the project's Docker Compose setup, commonly:

```text
http://localhost
```

Vite normally uses port `5173` for development. Docker Compose must expose the relevant port for browser access and hot reload.

If the page does not load, check the container status:

```bash
./vendor/bin/sail ps
```

Check the logs:

```bash
./vendor/bin/sail logs -f
```

## 12. Connect DataGrip to PostgreSQL

Open DataGrip and create a PostgreSQL data source.

Use the following connection settings when the default Compose port mapping is active:

| Setting | Value |
|---|---|
| Host | `127.0.0.1` |
| Port | `5432` |
| Database | `koko_db` |
| User | `sail` |
| Password | `password` |

Click **Test Connection** and download the driver if DataGrip requests it.

If port `5432` is already in use on Windows, change the forwarded port in your environment configuration and use that port in DataGrip. Keep `DB_PORT=5432` for Laravel's internal Docker connection.

## 13. Useful Docker Commands

Start the project:

```bash
./vendor/bin/sail up -d
```

Stop the project:

```bash
./vendor/bin/sail down
```

View running containers:

```bash
./vendor/bin/sail ps
```

View logs:

```bash
./vendor/bin/sail logs -f
```

Open a shell in the Laravel container:

```bash
./vendor/bin/sail shell
```

Run Laravel commands:

```bash
./vendor/bin/sail artisan migrate
./vendor/bin/sail artisan config:clear
./vendor/bin/sail artisan route:list
```

Rebuild the Docker image when Dockerfile or image dependencies change:

```bash
./vendor/bin/sail build
```

You generally do not need to rebuild the image for ordinary code changes.

## 14. Docker Cleanup and RAM Usage

Docker Desktop and WSL2 share the computer's physical RAM. Laravel, PostgreSQL, Vite and other containers all contribute to memory usage.

Stop containers when you are finished:

```bash
./vendor/bin/sail down
```

Inspect container memory usage:

```bash
docker stats
```

To remove unused Docker resources:

```bash
docker system prune
```

Review the prompt before confirming. This removes unused resources, but does not remove volumes by default.

**Warning:** Avoid `docker system prune --volumes` and `docker compose down -v` unless you intentionally want to delete relevant database volumes and their data.

If WSL uses too much memory, create or edit `.wslconfig` from Windows PowerShell:

```powershell
notepad "$env:USERPROFILE\.wslconfig"
```

Example settings for a PC with 16 GB RAM:

```ini
[wsl2]
memory=6GB
swap=2GB
```

Apply the settings from PowerShell:

```powershell
wsl --shutdown
```

This stops all WSL distributions. Restart Docker Desktop and Ubuntu afterward. Adjust the memory limit if your applications need more resources.

## 15. Troubleshooting

### Composer reports `php: not found`

Your Ubuntu terminal may be using a Windows Composer installation.

Check the executable paths:

```bash
which php
which composer
```

Paths under `/mnt/c/` indicate Windows executables. Use a Linux-native installation or run Composer through Sail after the Sail executable is available.

### Windows npm runs inside Ubuntu

Check:

```bash
which node
which npm
```

If the paths point to `/mnt/c/Program Files/nodejs`, the terminal is using Windows Node.js.

Use Sail for frontend commands:

```bash
./vendor/bin/sail npm install
./vendor/bin/sail npm run dev
```

Alternatively, install Linux-native Node.js in Ubuntu if you want to run npm directly in WSL.

### `vp: Permission denied`

The project uses Vite Plus, with the `vp` command supplied by the `vite-plus` dependency.

Inspect the executable:

```bash
ls -l node_modules/.bin/vp
cat node_modules/.bin/vp
```

If the executable lacks execute permission, correct it:

```bash
chmod +x node_modules/.bin/vp
```

If the error persists, reinstall dependencies in a consistent Linux or container environment.

### Laravel cannot connect to PostgreSQL

Check `.env`:

```dotenv
DB_CONNECTION=pgsql
DB_HOST=pgsql
DB_PORT=5432
```

Clear cached configuration:

```bash
./vendor/bin/sail artisan config:clear
```

Verify the database container is running:

```bash
./vendor/bin/sail ps
```

The host `pgsql` is the internal Docker service name. Use `127.0.0.1` only for connections originating from Windows applications such as DataGrip.

### Vite manifest not found

Build the frontend assets:

```bash
./vendor/bin/sail npm install
./vendor/bin/sail npm run build
```

For active development, run `npm run dev` through Sail instead.

### `/usr/bin/env: 'bash\r': No such file or directory`

This can occur when a script has Windows CRLF line endings.

Convert the affected script to Unix line endings:

```bash
sed -i 's/\r$//' vendor/bin/sail
```

Only convert the additional Sail script if it also has the same issue.

### Git reports `Author identity unknown`

Configure your Git identity:

```bash
git config --global user.name "Your Name"
git config --global user.email "YOUR_GITHUB_EMAIL"
```

### Git push fails with `src refspec main does not match any`

This usually means the local `main` branch has no commit yet.

Check:

```bash
git status
git branch
git log -1
```

Stage and commit your changes if appropriate, then push:

```bash
git add .
git commit -m "Initial commit"
git branch -M main
git push -u origin main
```

Review staged files before committing to ensure `.env`, credentials and generated secrets are excluded.

## 16. Daily Development Workflow

1. Start Docker Desktop.
2. Open Ubuntu.
3. Enter the project:

   ```bash
   cd ~/projects/koko
   ```

4. Start the containers:

   ```bash
   ./vendor/bin/sail up -d
   ```

5. Open VS Code:

   ```bash
   code .
   ```

6. Start Vite:

   ```bash
   ./vendor/bin/sail npm run dev
   ```

7. Develop using VS Code, test in your browser, and manage the database with DataGrip.
8. Stop Vite when finished, then stop the containers if you no longer need them.

## Notes

- Keep the active project in the WSL Linux filesystem.
- Use Sail for consistent PHP, PostgreSQL and Node.js execution.
- Keep `.env` and private keys out of version control.
- Use `docker compose down` without `-v` to preserve database volumes.
- Rebuild Docker images only when required, not for every code change.
