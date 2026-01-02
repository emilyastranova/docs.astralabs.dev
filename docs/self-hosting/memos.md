# Configuring Memos with Authentik SSO

This guide outlines the steps to configure Memos for Single Sign-On (SSO) with Authentik.

**Prerequisites:**

*   A running instance of Memos, accessible at a URL like `https://memos.mydomain.me`.
*   A running instance of Authentik, accessible at a URL like `https://auth.mydomain.me`.

## Authentik Configuration

First, you will need to configure Authentik to act as an OAuth2/OpenID provider for Memos.

1.  **Create a Provider:**
    *   In Authentik, create a new **OAuth2/OpenID Provider**.
    *   **Name:** `memos`
    *   **Authorization flow:** Select `default-provider-authorization-implicit-consent`.
    *   **Client type:** `Confidential`. Make a note of the **Client ID** and **Client Secret** as you will need them later.
    *   **Redirect URIs:** Add the following, substituting your actual domains and IPs:
        *   `https://memos.mydomain.me/auth/callback` (It's important to place your external domain first for the Authentik app launcher to function correctly).
        *   `http://<HOST_MACHINE_IP>:<HOST_MACHINE_PORT>/auth/callback` (The default Memos port is 5230).

2.  **Create an Application:**
    *   Create a new **Application**.
    *   **Name:** `Memos`
    *   **Slug:** `memos`
    *   **Provider:** Select the `memos` provider you created in the previous step.

3.  **Note Endpoint URLs:**
    *   Navigate back to the `memos` **Provider** settings.
    *   On the right side of the page, copy the **Authorization Endpoint**, **Token endpoint**, and **User endpoint** URLs. You will need these for the Memos configuration.

## Memos Configuration

Next, you will configure Memos to use the Authentik provider you just created.

1.  **Create an SSO Provider in Memos:**
    *   In Memos, navigate to **Settings > SSO** and click **Create**.
    *   **Template:** `Custom`
    *   **Name:** `Authentik`
    *   **Client ID:** Paste the Client ID from the Authentik provider.
    *   **Client Secret:** Paste the Client Secret from the Authentik provider.
    *   **Authorization Endpoint:** Paste the Authorization Endpoint URL from Authentik.
    *   **Token endpoint:** Paste the Token endpoint URL from Authentik.
    *   **User endpoint:** Paste the User endpoint URL from Authentik.
    *   **Scopes:** `openid email profile`
    *   **Identifier:** `preferred_username`
    *   **Display Name:** `name`
    *   **Email:** `email`

2.  **Disable Password Login (Optional):**
    *   Once you have confirmed that SSO is working correctly, you can navigate to **System settings** in Memos and disable password login for added security.