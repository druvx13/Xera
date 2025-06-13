# Xera Setup Guide for cPanel with Terminal Access

This guide will walk you through setting up and modernizing the Xera project on a cPanel hosting environment that provides terminal access.

**Prerequisites:**

*   Access to your cPanel account.
*   The "Terminal" feature available in your cPanel (usually in the "Advanced" section).
*   The project files already pushed to a GitHub repository.

## Step 1: Accessing the cPanel Terminal

1.  Log in to your cPanel account.
2.  Look for an icon named **"Terminal"** (often found in the "Advanced" or "Tools" section). Click on it.
3.  This will open a terminal window or interface directly in your browser.

## Step 2: Navigating to Your Project Directory

You'll need to navigate to the directory where you want to install Xera, or where it's already located if you've uploaded it manually. Common locations are `public_html` or a subdirectory within it.

*   **`ls`**: Lists files and directories in the current location.
*   **`cd <directory_name>`**: Changes directory. For example, `cd public_html` moves into the `public_html` folder.
*   **`cd ..`**: Moves up one directory level.
*   **`pwd`**: Shows your current directory path.

**Example:** If you want to install Xera in `public_html/xera`, you might do:
```bash
cd public_html
mkdir xera  # If the directory doesn't exist yet
cd xera
```

## Step 3: Getting the Project Files from GitHub

If you haven't uploaded the files yet, the easiest way is to clone the repository. If you have already uploaded them but need the latest changes (like the ones I committed), you'll `pull` the changes.

*   **To Clone (First time setup in this directory):**
    ```bash
    # Replace <your_github_repo_url> with the actual URL of your repository
    # e.g., https://github.com/username/Xera.git
    git clone <your_github_repo_url> .
    # The "." at the end clones it into the current directory
    ```

*   **To Pull Latest Changes (If already cloned/uploaded and it's a git repo):**
    Make sure you are in the main project directory (where `index.php`, `app/`, etc., are).
    ```bash
    git pull origin main # Or whatever your main branch is called (e.g., master, dev)
    ```

## Step 4: Checking for and Installing Composer

Composer is a dependency manager for PHP. We need it to install and update the PHP libraries Xera uses.

1.  **Check if Composer is already installed:**
    In the terminal, type:
    ```bash
    composer --version
    ```
    If it shows a version number, Composer is installed. Proceed to Step 5.

2.  **If Composer is not installed:**
    Many cPanel hosts don't have Composer installed system-wide but allow you to install it locally in your user directory. Here's a common method:
    *(Execute these commands one by one in your cPanel terminal, from your project's main directory or your home directory. If you install it in your home directory, you'll call it using `php ~/composer.phar` instead of just `composer`)*

    ```bash
    # Go to your home directory (optional, but a common place for local tools)
    # cd ~

    # Download the Composer installer script
    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

    # Verify the installer signature (optional but recommended - get current hash from https://composer.github.io/installer.html)
    # You would get the latest hash from the Composer website and replace 'YOUR_EXPECTED_HASH'
    # php -r "if (hash_file('sha384', 'composer-setup.php') === 'YOUR_EXPECTED_HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

    # Run the installer
    php composer-setup.php

    # Remove the installer script
    php -r "unlink('composer-setup.php');"
    ```
    This will download `composer.phar` in your current directory. Now, when you need to use Composer from this directory, you'll type `php composer.phar` instead of just `composer`. If you installed it in your home directory (`~`), you'd use `php ~/composer.phar`. For simplicity, the rest of this guide will use `composer`, but remember to adjust if you installed it locally as `composer.phar`.

    *   **Test local Composer:**
        ```bash
        # If installed in current directory:
        php composer.phar --version
        # If installed in home directory:
        # php ~/composer.phar --version
        ```

## Step 5: Installing/Updating PHP Dependencies with Composer

Now that you have the project files (with the updated `app/composer.json`) and Composer is available:

1.  Navigate to the **root directory of the Xera project** in your terminal (the directory that contains `index.php` and the `app` folder).
2.  Run the following command:
    ```bash
    # If composer is globally installed:
    composer update --working-dir=app/ --no-dev --optimize-autoloader

    # OR if you installed composer.phar locally in the project root:
    # php composer.phar update --working-dir=app/ --no-dev --optimize-autoloader

    # OR if you installed composer.phar in your home directory (~):
    # php ~/composer.phar update --working-dir=app/ --no-dev --optimize-autoloader
    ```
    *   `--working-dir=app/`: Tells Composer to look for `composer.json` inside the `app` directory.
    *   `--no-dev`: Skips installing development-only dependencies.
    *   `--optimize-autoloader`: Optimizes the autoloader for better performance.

    This command will read `app/composer.json`, download the specified versions of the libraries (`infinityfree/mofh-client: ^0.9.0` and `infinityfree/acmecore: ^3.1.1` and their dependencies), install them into the `app/vendor/` directory, and create/update the `app/composer.lock` file.

## Step 6: Manually Updating Frontend Libraries

The core files for some frontend libraries were replaced with placeholders in the Git repository. You'll need to download the actual library files and put them in place.

**General Process:**
1.  Download the "minified" version of the library where available (usually ends in `.min.js` or `.min.css`).
2.  Use the cPanel File Manager (or FTP/SFTP if you prefer, or `scp`/`rsync` via terminal if you're comfortable) to upload the downloaded file, replacing the placeholder file in the `assets/` directory.

Here are the libraries and their locations:

1.  **Tabler CSS** (Latest Stable Version)
    *   **Download from:** [Tabler GitHub Releases](https://github.com/tabler/tabler/releases) (Look for the latest stable release, download the `tabler.min.css` from the `dist/css` folder or the full package and extract it).
    *   **Replace:** `assets/default/css/tabler.min.css`
    *   **Also:** If the downloaded Tabler package includes other assets (like specific JS or fonts for Tabler itself), place them appropriately. Tabler's main CSS usually includes its necessary font imports.
    *   **Crucial:** After updating, review `assets/default/css/style.css` for any custom styles that might need adjustments to be compatible with the new Tabler version.

2.  **Font Awesome** (Latest Free Version - v6 or newer recommended)
    *   **Download from:** [Font Awesome Download Page](https://fontawesome.com/download) (Choose the "Free For Web" option).
    *   **Replace/Add CSS:**
        *   Upload `fontawesome-free-[version]-web/css/all.min.css` to `assets/default/css/all.min.css` (replacing the placeholder).
    *   **Replace/Add Webfonts:**
        *   Upload all files from the `fontawesome-free-[version]-web/webfonts/` directory into your `assets/default/webfonts/` directory.
    *   **Crucial - Update Icon Classes in HTML:** This is a manual code change you need to do. Font Awesome 5 (which was used) has syntax like `<i class="fa fa-user"></i>` or `<em class="fas fa-info"></em>`. Font Awesome 6 uses a different syntax, typically `<i class="fa-solid fa-user"></i>` or `<em class="fa-solid fa-info"></em>`.
        *   You will need to go through all template files in `template/default/` (and its subdirectories like `page/`, `form/`, `includes/`, `errors/`) and update these classes.
        *   **How to Find & Replace:**
            *   Use a text editor (like VS Code, Sublime Text, Notepad++) that can "Find in Files" or "Replace in Files" across multiple files if you download the `template` directory locally, make changes, and re-upload.
            *   Alternatively, you might be able to use `sed` or `awk` commands in the cPanel terminal if you are comfortable, but this is advanced and risky if not done carefully. **Manual editing or local find/replace is safer.**
            *   Search for patterns like `class="fa ` (and then ensure it's `fa fa-icon`), `class="fas ` , `class="far ` , `class="fab `.
            *   Replace them according to the Font Awesome 6 documentation.
                *   `fa fa-icon` generally becomes `fa-solid fa-icon` (for solid style).
                *   `fas fa-icon` becomes `fa-solid fa-icon`.
                *   `far fa-icon` becomes `fa-regular fa-icon`.
                *   `fab fa-icon` becomes `fa-brands fa-icon`.
            *   **Example:** `<i class="fa fa-users"></i>` becomes `<i class="fa-solid fa-users"></i>`.
            *   **Refer to:** The official [Font Awesome 5 to 6 Upgrade Guide](https://fontawesome.com/docs/web/setup/upgrade/whats-changed) for exact class mappings.
            *   This is the most labor-intensive part of the frontend update. Be careful and test thoroughly.

3.  **jQuery** (Latest Stable 3.x Slim Build)
    *   **Download from:** [jQuery Download Page](https://jquery.com/download/) (Look for the "slim minified" version of jQuery 3.x).
    *   **Replace:** `assets/default/js/jquery.slim.js`

4.  **ApexCharts** (Latest Stable Version)
    *   **Download from:** [ApexCharts GitHub Releases](https://github.com/apexcharts/apexcharts.js/releases) (Download `apexcharts.min.js` from the `dist` folder of the latest release).
    *   **Replace:** `assets/default/js/apexcharts.min.js`

## Step 7: Review Custom Styles

*   Open `assets/default/css/style.css` (e.g., using the cPanel File Manager's editor).
*   After updating Tabler, check if any of your custom styles (especially those for CKEditor or alerts) are broken or look different. You may need to adjust CSS selectors to match the new Tabler version's HTML structure or classes.

## Step 8: Testing

This is the most critical step.

1.  **PHP Error Reporting:** For testing, you might want to temporarily enable full PHP error reporting. You can often do this in cPanel's "MultiPHP INI Editor" or by adding these lines near the top of the main `index.php`:
    ```php
    // Add near the top of index.php
    // error_reporting(E_ALL);
    // ini_set('display_errors', 1);
    ```
    **Important: Remember to remove or comment these lines out for a live production site!**

2.  **Application Functionality:**
    *   Test every feature of Xera: login, registration, admin panel functions, user account functions, SSL, tickets, etc.
    *   Open your browser's developer console (usually F12) and check for JavaScript errors.
    *   Ensure all pages display correctly and all icons (Font Awesome) are visible.
    *   Test on different browsers if possible.

## Step 9: Final Checks

*   Once everything is working, ensure you removed any temporary debugging code (like PHP error reporting in `index.php`).
*   Your site should now be running on PHP 8 with updated libraries!

If you encounter specific errors during these steps (e.g., Composer errors, specific PHP errors after update), you may need to provide those error messages for further assistance.
