# Squirnox Password Manager

> Showcase repository. The full Squirnox source code is private. This README documents what the app does, how it works, and how it keeps your data safe. Screenshots coming soon.

## Overview

Squirnox is a secure, offline-first password manager designed to store your sensitive credentials locally within your browser. It prioritizes security and privacy by operating entirely offline, using strong encryption, and offering features like multi-profile support and a Secondary Password System (SPS) for critical entries.

## Features

*   **Offline Operation:** All data is stored locally in your browser's IndexedDB. No data is ever sent to any external server.
*   **Strong Encryption:** Uses AES-256-GCM for vault encryption and PBKDF2 for key derivation, ensuring your data is protected even if the storage is accessed.
*   **Multi-Profile Support:** Create and manage multiple independent profiles, each with its own master password and encrypted vault.
*   **Secondary Password System (SPS):** Protect highly sensitive entries with an additional, unique password (SPS password) that is never stored.
*   **Secure Backup & Import:** Export your encrypted vault for backup and import it back when needed. Backups are encrypted using your master password.
*   **Password Generation:** Generate strong, customizable passwords (length, character types).
*   **Clipboard Management:** Automatically clear copied passwords from the clipboard after a configurable timeout.
*   **Auto-Lock:** Automatically lock the vault after a period of inactivity.
*   **Theme Support:** Choose between dark (default) and light themes.
*   **Search Functionality:** Quickly find entries by name or username.
*   **Profile Wipe:** Securely and permanently delete a profile and all its associated data.

## Security Architecture

Squirnox employs several layers of security:

1.  **Master Password:** Each profile is protected by a unique master password. This password is **never stored directly**. 
2.  **Key Derivation (PBKDF2):** When you create a profile or log in, your master password and a unique, randomly generated salt (stored with the profile metadata) are used with PBKDF2 (Password-Based Key Derivation Function 2) with a high iteration count to derive a strong encryption key (Vault Key). This makes brute-forcing the master password extremely difficult.
3.  **Vault Encryption (AES-256-GCM):** The derived Vault Key is used with AES-256-GCM (Advanced Encryption Standard in Galois/Counter Mode) to encrypt and decrypt your vault data (entries and settings). GCM provides both confidentiality and data integrity.
4.  **Secondary Password System (SPS):** For critical entries, you can set an SPS password. This password is used *only* to encrypt/decrypt the specific password field of that entry using a key derived from the SPS password itself (again via PBKDF2). The SPS password is **never stored anywhere**, requiring you to enter it each time you need to access or modify the critical password.
5.  **Local Storage (IndexedDB):** All profile metadata (including the salt and encrypted vault) is stored locally in your browser's IndexedDB storage. This data remains on your computer and is isolated per browser profile.
6.  **In-Memory Protection:** Sensitive data like the master password and derived keys are kept in memory only while the vault is unlocked and are cleared upon locking or closing the application.

## Project Structure

```
/squirnox/
|-- index.html             # Main application HTML file (entry point)
|-- README.md              # This documentation file
|-- todo.md                # Development task checklist
|-- /assets/
|   |-- style.css          # CSS styles for the application
|   |-- favicon.ico        # (Optional) Application icon
|-- /src/
|   |-- crypto.js          # Core cryptographic functions (AES, PBKDF2)
|   |-- storage.js         # Handles interaction with IndexedDB for profiles/vaults
|   |-- ui.js              # Manages UI elements, views, and interactions
|   |-- main.js            # Main application logic, event handling, state management
```

## Setup and Usage

Squirnox is designed to run directly in your web browser without needing a web server.

1.  **Get the app:** Squirnox runs directly in your web browser without needing a web server. The source code is private, so the app files are not included in this showcase repo.
2.  **Open:** Open the `index.html` file in a modern web browser (like Chrome, Firefox, Edge, Safari).
3.  **Create Profile:** On the first launch (or if no profiles exist), you'll be prompted to create a new profile. Choose a unique profile name and a **strong, memorable master password**. Confirm the password.
4.  **Login:** Select your profile and enter your master password to unlock the vault.
5.  **Use:** Add, edit, search, and manage your password entries.

**Important:** Since Squirnox stores data in your browser's IndexedDB, clearing your browser's site data for the `file://` protocol (or the specific origin if hosted) **will delete your vaults**. Always back up your vaults regularly using the export feature.

## Multi-Profile Management

*   **Creating:** Use the "Create Profile" section on the initial screen. Provide a name and a strong master password.
*   **Selecting:** Click the button corresponding to the profile you wish to access.
*   **Isolation:** Each profile is entirely separate, with its own master password, salt, and encrypted vault.

## SPS (Secondary Password System)

*   **Purpose:** Provides an extra layer of security for your most critical passwords (e.g., primary email, bank accounts).
*   **Enabling:** When adding or editing an entry, check the "Mark as Critical" box.
*   **Setting SPS Password:** You will be prompted to enter an SPS password specifically for that entry. **You must remember this password.** It is used to encrypt the entry's password field and is *never stored*.
*   **Accessing:** To view or copy the password of a critical entry, you will need to enter the correct SPS password for that specific entry.
*   **Responsibility:** Losing the SPS password for an entry means the password for that entry is irrecoverably lost.

## Backup and Import/Export

*   **Export:** 
    *   Go to the "Settings" view.
    *   Click "Export Vault".
    *   A JSON file (`squirnox_backup_[profileName]_[timestamp].json`) containing your *encrypted* vault data and the necessary salt will be downloaded.
    *   **Keep this file safe!** Anyone with this file and your master password can decrypt your vault.
*   **Import:**
    *   Go to the "Settings" view.
    *   Click the "Import Vault" label/button and select a previously exported backup JSON file.
    *   The application will decrypt the backup using your *current* master password and the salt *from the backup file*.
    *   Entries from the backup will be merged into your current vault. Entries with the same name and username as existing entries will be skipped to avoid simple duplicates.

## Settings

Access the settings view via the "Settings" button in the main vault view.

*   **Theme:** Switch between Dark and Light mode.
*   **Clear Clipboard After:** Set the timeout (in seconds) after which copied passwords will be automatically cleared from the clipboard.
*   **Auto-Lock After:** Set the inactivity timeout (in minutes) after which the vault will automatically lock.
*   **Backup & Recovery:** Access Export and Import functions.
*   **Danger Zone:** Access the Wipe Profile function.

## Wipe Profile Feature

*   **Location:** Found in the "Danger Zone" section of the Settings view.
*   **Function:** Permanently deletes the currently logged-in profile, including its metadata (salt) and encrypted vault from IndexedDB.
*   **Warning:** This action is **irreversible**. You will be asked for confirmation multiple times. Ensure you have backups if you might need the data later.

## Troubleshooting

*   **Cannot Create Profile/Save Vault:** 
    *   Ensure your browser supports IndexedDB and it's enabled.
    *   Check if your browser's private/incognito mode is blocking storage access.
    *   Check browser console (usually F12 key) for specific error messages.
*   **Login Failed (Incorrect Password):** Double-check your master password. It's case-sensitive.
*   **Cannot Decrypt Backup:** Ensure you are importing into the correct profile and using the master password that was active *when the backup was created*.
*   **Entries Missing After Clearing Browser Data:** Squirnox stores data locally. Clearing site data for `file://` URLs or the specific origin will delete vaults. Restore from a backup.
*   **UI Issues/Buttons Not Working:** Try refreshing the page (Ctrl+R or Cmd+R). Check the browser console for errors.

## Security Considerations

*   **Master Password Strength:** Your primary defense. Use a long, complex, and unique master password. Do not reuse it anywhere else.
*   **SPS Password Management:** Remember your SPS passwords. There is no recovery mechanism for them.
*   **Backups:** Regularly export your vault and store the backup file securely (e.g., on an encrypted USB drive). Protect the backup file as it contains your encrypted data.
*   **Physical Security:** Protect the computer where you use Squirnox from unauthorized access.
*   **Browser Security:** Keep your browser updated to protect against vulnerabilities.
*   **Phishing/Malware:** Be cautious of malware that could potentially log keystrokes or access browser data.

