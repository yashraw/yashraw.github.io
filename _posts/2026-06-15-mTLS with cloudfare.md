---
title: Accessing Home Assistant Securely
tags: selfhosting
style: fill
color: dark
description: Access your instance of Home Assistant securily with mTLS Setup with Cloudfare
date: 2026-06-15
author: Yash Rathod
---


original - https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197

## [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-secure-remote-and-local-ha-access-via-cloudflare-tunnel-mtls-and-nginx-proxy-manager-1)Secure Remote and Local HA Access Via Cloudflare Tunnel + mTLS and Nginx Proxy Manager

This guide walks through setting up a Cloudflare Tunnel on Home Assistant OS using the Cloudflared app, then securing it with mutual TLS (mTLS) using Cloudflare-issued client certificates and a WAF custom rule — no Enterprise plan required. It also briefly covers configuring Nginx Proxy Manager for local SSL/TLS.

## [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-why-do-this-what-are-the-benefits-2)Why do this? What are the benefits?

-   **Speed** - When outside your LAN, connections route through the closest Cloudflare Point of Presence (PoP). These PoPs allow them to reach approximately 95% of the world's internet-connected population within 50 milliseconds.
    
-   **Resiliancy** - Cloudflare operates in over **335 cities** across more than 125 countries, including mainland China.
    

___

> **Before you start:**
> 
> If you also want local (on-network) access to Home Assistant without routing through Cloudflare, use **separate hostnames** for external and internal access — for example `ha.yourdomain.com` externally and `lan.ha.yourdomain.com` internally. Split-brain DNS (same hostname resolving to different IPs inside vs outside the network) does **not** work reliably with Cloudflare-fronted hostnames, because of DNS-over-HTTPS (DoH) behavior on iOS, macOS Safari, and on some network appliances. See **Part 8** for the split-URL architecture and **Appendix A** for the full cautionary tale.

___

## [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-prerequisites-3)Prerequisites

-   A domain name with DNS managed by Cloudflare
-   A Cloudflare account (Free plan or above)
-   Home Assistant OS running with Supervisor/app-on support
-   Access to the Home Assistant UI and the ability to edit `configuration.yaml`
-   OpenSSL installed on your local machine (for `.p12` export)
-   (Optional, for the split-URL architecture in Part 8) Nginx Proxy Manager app-on or another local reverse proxy

___

## [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-part-1-install-and-configure-the-cloudflared-app-4)Part 1: Install and Configure the Cloudflared App

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-h-11-add-the-repository-5)1.1 Add the Repository

1.  In Home Assistant, go to **Settings > apps > App Store**.
2.  Click the **three-dot menu** (top right) and select **Repositories**.
3.  Add the following repository URL:
    
    ```
    https://github.com/homeassistant-apps/repository
    ```
    
4.  Click **Add**, then close the dialog. The "Cloudflare Tunnel Client" app should now appear in the store.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-h-12-install-the-app-6)1.2 Install the App

1.  Find **Cloudflare Tunnel Client** in the app store and click **Install**.
2.  After installation, do **not** start it yet — configure it first.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-h-13-choose-a-tunnel-mode-7)1.3 Choose a Tunnel Mode

The app supports two modes:

-   **Local tunnel (managed by the app):** The app handles tunnel creation and DNS. Simpler, but limited to exposing Home Assistant on one hostname.
-   **Remote tunnel (managed in Cloudflare dashboard):** You create the tunnel in the Cloudflare Zero Trust dashboard and pass the token to the app. This gives you full control over multiple public hostnames and ingress rules. **Use this mode for mTLS setups.**

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-h-14-create-a-remote-tunnel-in-cloudflare-8)1.4 Create a Remote Tunnel in Cloudflare

1.  Log in to the [Cloudflare Zero Trust dashboard](https://one.dash.cloudflare.com/).
2.  Go to **Networks > Connectors**.
3.  Click **Create a tunnel** and choose **Cloudflared** as the connector type.
4.  Give it a name (e.g., `homeassistant`).
5.  On the connector install page, copy the **tunnel token** — you'll need it for the app config.
6.  Under **Public Hostnames**, add a route:
    -   **Subdomain:** e.g., `ha` (will become `ha.yourdomain.com`)
    -   **Domain:** select your Cloudflare-managed domain
    -   **Service:** `http://homeassistant:8123` (the internal Docker hostname)
        -   If that doesn't work, try `http://172.30.32.1:8123`
7.  Save the tunnel.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-h-15-configure-the-app-9)1.5 Configure the app

1.  Go to the Cloudflared app **Configuration** tab in Home Assistant.
2.  Set the tunnel token:
    
    ```
    tunnel_token: "eyJhIjoixxxxxxx..."
    ```
    
3.  Make sure the tunnel name matches what was used in the Cloudflare dashboard (not sure if this is necessary)
4.  Leave other options at defaults unless you have specific needs.
5.  Save the configuration.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-h-16-update-home-assistant-configurationyaml-10)1.6 Update Home Assistant configuration.yaml

Home Assistant must trust the Cloudflared proxy. Add the following to your `configuration.yaml`:

```
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 172.30.33.0/24
```

After saving, restart Home Assistant (not just reload — a full restart is needed for `http` config changes).

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-h-17-start-the-app-11)1.7 Start the app

1.  Go back to the Cloudflared app and click **Start**.
2.  Check the **Log** tab to confirm the tunnel connects successfully. You should see messages about the tunnel being registered.
3.  Test basic access by navigating to `https://ha.yourdomain.com` in a browser. You should see your Home Assistant login page.

___

## [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-part-2-verify-cloudflare-ssltls-settings-12)Part 2: Verify Cloudflare SSL/TLS Settings

Before moving on to mTLS, confirm that standard HTTPS is working for your tunnel hostname. All three of the following settings are required.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-h-21-set-ssltls-encryption-mode-to-full-13)2.1 Set SSL/TLS Encryption Mode to Full

1.  In the Cloudflare dashboard, select your domain.
2.  Go to **SSL/TLS > Overview**.
3.  Set the encryption mode to **Full**.

Do **not** use "Off" or "Flexible" — these will cause connection failures or redirect loops. "Full" is correct because Cloudflare terminates TLS at the edge and forwards traffic to Home Assistant over the tunnel as plain HTTP. Do not use "Full (Strict)" unless you have a valid origin certificate installed on your HA instance.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-h-22-enable-always-use-https-14)2.2 Enable Always Use HTTPS

1.  Go to **SSL/TLS > Edge Certificates**.
2.  Enable **Always Use HTTPS**.

This ensures all HTTP requests to your hostname are automatically redirected to HTTPS.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-h-23-ensure-your-hostname-has-a-valid-edge-certificate-15)2.3 Ensure Your Hostname Has a Valid Edge Certificate

Cloudflare's free Universal SSL certificate covers `yourdomain.com` and `*.yourdomain.com`, but it does **not** cover deeper subdomains like `ha.home.yourdomain.com`. If your tunnel hostname is two or more levels deep, you need an additional certificate and there could be a $10/mo fee to use the Advanced Certificate Manager.

Check for a warning on the DNS record that says "This hostname is not covered by a certificate." If you see it:

1.  Go to **SSL/TLS > Edge Certificates**.
2.  Click **Order Advanced Certificate**.
3.  Add both the wildcard and the base for your subdomain level:
    -   `*.home.yourdomain.com`
    -   `home.yourdomain.com`
4.  Complete the order. Cloudflare will provision the certificate (this is available on the free plan).

Once provisioned, verify that `https://ha.yourdomain.com` loads your Home Assistant login page before proceeding.

___

## [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-part-3-create-client-certificates-using-cloudflares-ca-16)Part 3: Create Client Certificates Using Cloudflare's CA

Instead of generating your own CA, Cloudflare acts as the certificate authority and issues client certificates directly from the dashboard. This works on all plans (Free and above).

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-h-31-create-a-client-certificate-17)3.1 Create a Client Certificate

1.  In the Cloudflare dashboard, select your domain.
2.  Go to **SSL/TLS > Client Certificates**.
3.  Click **Create Certificate**.
4.  Cloudflare generates a certificate signed by a Cloudflare-managed, account-level root CA.
5.  Choose the key type and validity period:
    -   RSA or ECDSA (RSA 2048 is fine for broad device compatibility)
    -   Validity: 1 year, 2 years, or 10 years
6.  Cloudflare displays the **certificate** and **private key**.

> **Important:** Copy both the certificate and private key immediately. The private key is shown only once and cannot be retrieved later.

7.  Save them to files on your local machine:
    -   `client-cert.pem` — the certificate
    -   `client-key.pem` — the private key

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-h-32-repeat-for-additional-devices-18)3.2 Repeat for Additional Devices

Create a separate certificate for each device that needs access. This makes revocation straightforward — you can revoke a single device's certificate without affecting the others.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-h-33-export-as-p12-pkcs12-19)3.3 Export as .p12 (PKCS#12)

Browsers and mobile devices import `.p12` files for client certificate authentication. On your local machine, run:

```
openssl pkcs12 -export \
  -out client.p12 \
  -in client-cert.pem \
  -inkey client-key.pem \
  -name "HA Client Certificate"
```

You will be prompted to set an export password. Remember this — you'll need it when importing on your devices.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-h-34-file-summary-20)3.4 File Summary

| File | Purpose | Keep Secret? |
| --- | --- | --- |
| `client-cert.pem` | Client certificate (from Cloudflare) | No |
| `client-key.pem` | Client private key (from Cloudflare) | Yes — save it, Cloudflare won't show it again |
| `client.p12` | Bundled cert + key for device import | Yes — distribute to trusted devices only |

___

## [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-part-4-enable-mtls-and-enforce-with-a-waf-rule-21)Part 4: Enable mTLS and Enforce with a WAF Rule

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-h-41-associate-the-hostname-with-client-certificates-22)4.1 Associate the Hostname with Client Certificates

1.  In the Cloudflare dashboard, go to your domain's **SSL/TLS > Client Certificates**.
2.  In the **Hosts** section of the Client Certificates card, click **Edit**.
3.  Add `ha.yourdomain.com` (the hostname your tunnel uses).
4.  Save.

This tells Cloudflare to request a client certificate from any browser or device connecting to that hostname.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-h-42-create-a-waf-custom-rule-to-block-requests-without-a-valid-certificate-23)4.2 Create a WAF Custom Rule to Block Requests Without a Valid Certificate

1.  Go to **Security > WAF > Custom Rules**.
2.  Click **Create rule**.
3.  Configure the rule:
    -   **Rule name:** `Require mTLS for Home Assistant`
    -   **Expression (edit expression):**
        
        ```
        (http.host eq "ha.yourdomain.com")
        and (not cf.tls_client_auth.cert_verified)
        and (not starts_with(http.request.uri.path, "/api/webhook/"))
        ```
        
    -   **Action:** `Block`
4.  Deploy the rule.

Any request to `ha.yourdomain.com` that does not present a valid client certificate signed by your account's Cloudflare-managed CA will now receive a `403 Forbidden` response — **except** requests to the `/api/webhook/` path, which are allowed through without a client cert.

**Why the webhook exclusion is required.** The Home Assistant Companion App posts to `/api/webhook/<token>` for sensor updates, location updates, and actionable notification callbacks. These POSTs are made from a background `URLSession` on iOS (and the equivalent on Android) that does not share the foreground app's access to the installed client identity, so the cert is never presented on the TLS handshake. Without the `/api/webhook/` exclusion, every one of those POSTs is blocked by the mTLS rule, and the Companion App fails to save or verify the external URL with the error:

> **Error Saving URL — Unacceptable status code 403**

The `/api/webhook/` path is safe to exempt because HA webhook URLs contain a long random secret token that authenticates the caller. Treat those URLs as bearer secrets — do not paste them in screenshots, logs, shared backups, or issue trackers.

> **Tip:** Cloudflare's **"Create mTLS Rule"** template button on the Client Certificates page generates the basic form of this rule. You will still need to edit the expression afterward to add the `/api/webhook/` exclusion.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-h-43-test-the-configuration-24)4.3 Test the Configuration

1.  **Without the certificate installed:** Navigate to `https://ha.yourdomain.com` in a browser. You should be blocked with a 403 error.
2.  **With the certificate installed:** After importing the `.p12` (see Part 5), the browser should prompt you to select the client certificate. After selecting it, you should see your Home Assistant login page.
3.  **Webhook path spot-check:** With wifi off on your phone (forcing the Companion App onto cellular), re-verify the external URL in Settings → Companion App → Connection. It should save without the 403 error. In Cloudflare **Security > Events**, filter by `Path contains /api/webhook/` — you should see POSTs with **Mitigation: Not mitigated**.

___

## [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-part-5-install-the-p12-certificate-on-your-devices-25)Part 5: Install the .p12 Certificate on Your Devices

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-macos-26)macOS

1.  Double-click the `.p12` file — it opens in **Keychain Access**.
2.  Enter the export password you set earlier.
3.  The certificate is added to your login keychain. For Safari to present the cert reliably, you may need to drag it into the **System** keychain (Safari sometimes does not pick up client identities from the login keychain in newer macOS versions).
4.  When you visit `ha.yourdomain.com` in Safari or Chrome, the browser will prompt you to select the client certificate.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-ios-ipados-27)iOS / iPadOS

1.  AirDrop or email the `.p12` file to your device.
2.  Open the file — you'll be directed to **Settings > General > VPN & Device Management**.
3.  Tap **Install**, enter your device passcode, then enter the `.p12` export password.
4.  The profile is now installed. Safari will automatically present the certificate when visiting `ha.yourdomain.com`, and the HA Companion App's foreground `WKWebView` requests will pick it up as well.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-windows-28)Windows

1.  Double-click the `.p12` file to open the **Certificate Import Wizard**.
2.  Choose **Current User** as the store location.
3.  Enter the export password.
4.  Let Windows automatically select the certificate store (or choose **Personal**).
5.  Complete the wizard. Chrome and Edge will use this certificate automatically.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-android-29)Android

1.  Transfer the `.p12` file to the device.
2.  Go to **Settings > Security > Encryption & credentials > Install a certificate**.
3.  Select **VPN and app user certificate**.
4.  Browse to the `.p12` file, enter the password, and give it a name.
5.  When visiting `ha.yourdomain.com` in Chrome, you'll be prompted to select the certificate.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-firefox-all-platforms-30)Firefox (all platforms)

Firefox uses its own certificate store, separate from the OS:

1.  Go to **Settings > Privacy & Security > Certificates > View Certificates**.
2.  Under **Your Certificates**, click **Import**.
3.  Select the `.p12` file and enter the password.

___

## [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-part-6-home-assistant-companion-app-setup-31)Part 6: Home Assistant Companion App Setup

The Home Assistant Companion App supports mTLS natively. Once the client certificate is installed at the OS level, the app will present it automatically during the TLS handshake — with one important caveat about background requests (see Part 4.2).

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-ios-ipados-32)iOS / iPadOS

1.  Install the `.p12` profile on your device as described in Part 5 (iOS / iPadOS section).
2.  Open the Companion App and go to **Settings > Connection**.
3.  Set the **External URL** to `https://ha.yourdomain.com`.
4.  If you also use a local URL (recommended — see Part 8), set the **Internal URL** to `https://lan.ha.yourdomain.com` or whatever you chose.
5.  The app will use the installed client certificate when connecting over the external URL.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-android-33)Android

1.  Install the `.p12` certificate on your device as described in Part 5 (Android section).
2.  Open the Companion App and go to **Settings > Connection**.
3.  Set the **External URL** to `https://ha.yourdomain.com`.
4.  When the app connects, Android will prompt you to select the client certificate. Choose the one you installed.

___

## [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-part-7-maintenance-and-revocation-34)Part 7: Maintenance and Revocation

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-revoking-a-certificate-35)Revoking a Certificate

If a device is lost or compromised:

1.  Go to **SSL/TLS > Client Certificates** in the Cloudflare dashboard.
2.  Find the certificate for the compromised device.
3.  Click **Revoke**.
4.  The certificate is immediately invalidated. The WAF rule will block requests using that certificate going forward.

This is a significant advantage of using Cloudflare-issued certificates — you can revoke individual certificates without affecting other devices.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-creating-replacement-certificates-36)Creating Replacement Certificates

Follow the same process in Part 3 to create a new certificate for a replacement device. Each certificate is independent.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-certificate-limits-37)Certificate Limits

Cloudflare has a limit on the number of client certificates per account. If you hit the limit, revoke and delete any certificates that are no longer in use before creating new ones.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-monitoring-38)Monitoring

-   Check tunnel health in the Cloudflare Zero Trust dashboard under **Networks > Tunnels**.
-   Review blocked requests under **Security > Events** to see mTLS enforcement in action. Filter by `Action = Block` and `Host = ha.yourdomain.com` to see only the enforcement hits.
-   The WAF analytics will show how many requests are being blocked by the mTLS rule.

___

## [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-part-8-recommended-architecture-separate-external-and-internal-urls-39)Part 8: Recommended Architecture — Separate External and Internal URLs

For anyone on a home network that serves both on-LAN and remote clients, use **two separate hostnames**: one that goes through the Cloudflare Tunnel (with mTLS), and one that stays on the LAN and resolves directly to Home Assistant via a local reverse proxy.

**Example:**

| URL | Path | Purpose |
| --- | --- | --- |
| `https://ha.yourdomain.com` | Cloudflare edge → Cloudflare Tunnel → HA | External access from outside the home network. mTLS-enforced. |
| `https://lan.ha.yourdomain.com` | LAN client → local reverse proxy → HA | Internal access from devices on the home network. No Cloudflare, no mTLS. |

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-why-two-urls-40)Why two URLs

-   **Reliability.** Split-brain DNS (same hostname, different answers depending on the client's location) fails in subtle ways when the hostname is fronted by Cloudflare. See Appendix A for the full story — short version: modern clients use DoH, which bypasses your local DNS, so the "internal" answer never reaches them.
-   **Performance.** Local clients stay on the LAN. No trip out to the Cloudflare edge and back through a tunnel.
-   **Debuggability.** When something breaks you know exactly which path is at fault — the hostname tells you whether it's going through Cloudflare or not.
-   **Security boundary is clearer.** mTLS is enforced on `ha.yourdomain.com`. `lan.ha.yourdomain.com` has no mTLS because it's only reachable from inside the LAN — that's the access control.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-setting-up-the-local-url-41)Setting up the local URL

1.  Install the **Nginx Proxy Manager** app in Home Assistant (or use any local reverse proxy you prefer — Caddy, Traefik, etc.).
2.  In NPM, create a Proxy Host:
    -   **Domain name:** `lan.ha.yourdomain.com`
    -   **Scheme:** `http`
    -   **Forward hostname / IP:** `homeassistant`
    -   **Forward port:** `8123`
    -   **Websockets support:** **enabled** (required for HA's frontend)
    -   **Block common exploits:** optional
3.  On the **SSL** tab of the proxy host, request a Let's Encrypt certificate for `lan.ha.yourdomain.com`. NPM can complete the DNS-01 challenge against Cloudflare directly — use the Cloudflare API token option.
4.  In **Cloudflare DNS**, create a DNS-only (grey-cloud) A record:
    -   **Name:** `lan.ha`
    -   **Content:** the LAN IP of your Home Assistant host (e.g., `10.10.21.221`)
    -   **Proxy status:** DNS only — **not** proxied through Cloudflare
5.  Optionally create a matching AAAA record if you run IPv6 on the LAN. Use a static ULA or link-local IPv6 assigned to the HA host.
6.  From a LAN client, confirm `https://lan.ha.yourdomain.com` loads HA directly.

Because the record is DNS-only, external clients resolving `lan.ha.yourdomain.com` will also get the LAN IP — but they can't route to it, so it's effectively internal-only. This is acceptable; it's not a secret, and the LAN-only reachability is enforced by your router, not by DNS visibility.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-update-the-companion-app-42)Update the Companion App

-   **External URL:** `https://ha.yourdomain.com`
-   **Internal URL:** `https://lan.ha.yourdomain.com`

The app will automatically use the internal URL when connected to your home wifi (it compares the SSID) and the external URL otherwise.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-update-the-http-block-in-home-assistant-43)Update the `http:` block in Home Assistant

Since requests will now come from both the Cloudflared app and NPM, both need to be in `trusted_proxies`:

```
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 172.30.33.0/24   # Cloudflared app
```

Restart HA after changes to the `http:` block.

___

## [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-troubleshooting-44)Troubleshooting

| Issue | Solution |
| --- | --- |
| Browser doesn't prompt for certificate | Confirm the hostname is listed under SSL/TLS > Client Certificates > Hosts. Clear browser cache and restart the browser. |
| 403 even with cert installed | Verify the `.p12` was built from the correct cert/key pair. Check that the certificate hasn't been revoked in the Cloudflare dashboard. Try a different browser to rule out cert store issues. On macOS, try moving the client identity from the login keychain to the System keychain for Safari. |
| **Companion App: "Error Saving URL — Unacceptable status code 403"** | The app's background webhook POST is being blocked by the mTLS WAF rule. Add `and (not starts_with(http.request.uri.path, "/api/webhook/"))` to the rule expression. See Section 4.2. |
| Companion App saves the URL but notifications don't arrive, sensors don't update, location isn't posting | Same root cause as the 403 above — the webhook POSTs from the app's background `URLSession` don't present the client cert. Apply the `/api/webhook/` exclusion in Section 4.2. |
| Tunnel not connecting | Check the Cloudflared app logs in HA. Verify the tunnel token is correct. Ensure HA can reach the internet. |
| HA shows "400 Bad Request" or IP ban after adding NPM / Cloudflared | `trusted_proxies` is missing the proxy's subnet. Add both the Cloudflared and NPM app subnets to `http.trusted_proxies` in `configuration.yaml` and fully restart HA. |
| Companion App can't connect | Verify the `.p12` is installed at the OS level. Try accessing the URL in the device's browser first to confirm the cert works there. Then verify the WAF rule exempts `/api/webhook/`. |
| "Maximum number of certificates reached" | Delete revoked or unused certificates from SSL/TLS > Client Certificates before creating new ones. |
| `cf.tls_client_auth.cert_verified` is always false | Ensure the hostname is added to the Hosts list on the Client Certificates page. Without this, Cloudflare won't request or validate client certs. |
| **On-LAN clients get the Cloudflare "you have been blocked" page when hitting the external hostname** | You are hitting the split-brain DNS failure mode. The client's DNS is going through DoH and resolving to Cloudflare's public IPs instead of your LAN. The reliable fix is to move to the split-URL architecture in Part 8. See Appendix A for the details. |
| **macOS Safari intermittently shows the Cloudflare block page for a local hostname** | macOS system DNS in recent versions uses encrypted DNS paths that ignore `/etc/hosts` in some situations and ignore your LAN's local DNS server entirely. There is no user-facing toggle to disable this for a specific domain. Use the separate internal URL in Part 8 instead of trying to make split-brain DNS work. |
| **Chrome ignores local DNS for a Cloudflare-fronted hostname** | Chrome has its own DNS-over-HTTPS. Disable it at `chrome://settings/security` → **Use secure DNS = Off**. This makes Chrome use the OS resolver, which honors your LAN DNS. Note: this only mitigates Chrome; it does not fix Safari, iOS apps, or any native app on macOS. |
| **HA Companion App on iOS always takes the Cloudflare path for the external URL, even on-LAN** | iOS has no per-app or per-domain DoH override. When the OS resolves a Cloudflare-hosted name, it may use encrypted DNS that bypasses your LAN's DNS server. Accept it and use the split-URL architecture in Part 8 — the app will pick the internal URL automatically when on your home wifi. |

___

## [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-security-notes-45)Security Notes

-   **Save your private keys.** Cloudflare only shows the private key once at creation time. If you lose it, you'll need to revoke that certificate and create a new one.
-   **Use one certificate per device.** This makes revocation clean — you only revoke the compromised device's cert.
-   **The WAF rule is your enforcement layer.** Without it, Cloudflare will request a client cert but won't block requests that don't provide one. The WAF rule with `not cf.tls_client_auth.cert_verified` is what actually enforces access.
-   **Webhook path is exempted from mTLS.** The `/api/webhook/<token>` path is excluded from the WAF rule so the Companion App's background requests can post sensor and location updates. HA webhook URLs contain a long random secret that authenticates the caller. Treat those URLs as bearer secrets — do not include them in screenshots, logs, or shared backups.
-   **Consider shorter validity periods** (1 year) for devices that are more likely to be lost or compromised.
-   **HA login is still required.** mTLS controls who can reach your HA instance at the network level. Users still need valid HA credentials to log in. This is defense in depth.
-   **The internal URL is protected by your LAN, not by Cloudflare.** If an attacker is already on your home network, they can reach `lan.ha.yourdomain.com` without a client certificate. Harden your wifi (WPA3 or strong WPA2, guest network isolation, etc.) accordingly.

___

## [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-appendix-a-why-split-brain-dns-fails-with-cloudflare-a-cautionary-tale-46)Appendix A: Why Split-Brain DNS Fails with Cloudflare — A Cautionary Tale

This appendix documents what was tried, what broke, what was mitigated, and what turned out to be unfixable. It exists so you can skip rediscovering all of this yourself.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-the-goal-47)The goal

Run a single hostname — `ha.yourdomain.com` — that:

-   Resolves to the **LAN IP of the Home Assistant host** when the client is on the home network (fast, direct, no tunnel, no mTLS).
-   Resolves to the **Cloudflare edge** when the client is off-network (mTLS-protected via the tunnel).

This is the textbook "split-brain DNS" or "split-horizon DNS" pattern, and it works fine for hostnames that are _not_ fronted by Cloudflare. With a Cloudflare-fronted hostname, it fails.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-what-was-set-up-48)What was set up

-   A local DNS record on the UniFi Dream Machine Pro (UDMP) pointing `ha.yourdomain.com` to the HA host's LAN IPv4.
-   The public Cloudflare DNS record for `ha.yourdomain.com` pointing to the Cloudflare Tunnel (orange-cloud proxied).
-   A working Cloudflare Tunnel + mTLS rule (as described in Parts 1–4 of this guide).
-   A valid client certificate installed on the Mac and on an iPhone.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-symptoms-49)Symptoms

-   On the Mac, Safari, Chrome, and the HA Companion App would sometimes show the Cloudflare "Sorry, you have been blocked" page when opening `ha.yourdomain.com` from inside the home network.
-   On iPhone it was worse — the HA Companion App couldn't initially connect to the external URL at all from on-LAN cellular-off.
-   The same hostname worked fine from other clients some of the time, and sometimes stopped working after a browser cache clear.
-   `ping` and `curl` from a Mac terminal resolved to the LAN IP correctly. Browsers and native apps did not.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-root-cause-1-ipv6-aaaa-records-50)Root cause #1 — IPv6 AAAA records

Cloudflare automatically returns AAAA (IPv6) records for proxied hostnames that point at Cloudflare's anycast IPv6 addresses (`2606:4700::...`). Modern browsers implement RFC 8305 ("happy eyeballs") and prefer IPv6 when both IPv4 and IPv6 are available. The local DNS server (UDMP) was returning only the A record for the LAN IPv4 address. The AAAA record came back from public DNS, pointed at Cloudflare, and the browsers preferred it over the local A record.

**Mitigation:** Added a local AAAA record on the UDMP pointing the hostname to a local IPv6 address, and assigned a static IPv6 address to the HA host (HA OS does not support setting static IPv6 via DHCPv6 on the UDMP, so the static address was set on the HA host itself via `ha network update`).

This helped but did not fully fix it — traffic was still reaching Cloudflare.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-root-cause-2-dns-over-https-doh-bypassing-local-dns-51)Root cause #2 — DNS-over-HTTPS (DoH) bypassing local DNS

Modern clients do not necessarily use the DNS server advertised by DHCP. Instead, they send their DNS queries to a public resolver over encrypted HTTPS or TLS:

-   **Chrome** ships with its own built-in DoH enabled by default (`chrome://settings/security` → "Use secure DNS"). It resolves through its configured provider, bypassing the UDMP entirely.
-   **macOS Safari / system resolver**. Recent macOS versions can route DNS through encrypted transports opaquely. There is **no user-facing setting to turn this off for a specific domain**. Safari's use of the system resolver means the result of the encrypted lookup is what Safari sees — which is the public Cloudflare record, not the local one.
-   **iOS Home Assistant Companion App** uses the system resolver. iOS has the same encrypted-DNS behavior and the same lack of a per-domain override. There is no app-level setting to turn DoH off for the HA app.
-   **The UDMP itself** had UniFi CyberSecure's **Encrypted DNS** feature set to **Predefined: Cloudflare**. This meant the router's own upstream DNS lookups for public names were going over DoH to Cloudflare — so even the network-wide DNS path was encrypted and bypassing any hope of interception, and the router received the public answer for its own cache.

When DoH is in play, the "local DNS override" you configured on your router is simply not consulted.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-what-was-mitigated-partially-52)What was mitigated, partially

-   **Chrome** can be mitigated: go to `chrome://settings/security` and set **Use secure DNS** to **Off**. Chrome will then fall back to the OS resolver, which (on macOS, if the OS is not also doing its own DoH for that domain) honors your LAN DNS. This worked for Chrome specifically.
-   **UDMP** can have its DoH setting disabled under **Settings → Security → CyberSecure → Encrypted DNS → None**. This stops the _router_ from doing DoH but does not stop the individual client devices from doing their own.
-   **`/etc/hosts`** on macOS can override DNS for specific hostnames and does work for some clients, but Safari's behavior was inconsistent — it would honor `/etc/hosts` sometimes and ignore it others. A fresh Safari cache clear would often re-break it. The leading hypothesis is that Safari is routing certain lookups through an OS-level encrypted DNS path that does not consult `/etc/hosts`.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-what-could-not-be-mitigated-53)What could not be mitigated

-   **Safari on macOS.** No way to force it off the encrypted DNS path for a specific domain. No user-facing toggle. `/etc/hosts` is unreliable.
-   **HA Companion App on iOS.** Same story — no per-app or per-domain override for encrypted DNS. The OS resolver decides, and the OS resolver will often return the Cloudflare public answer.
-   **Any other native macOS or iOS app** that uses the system resolver. Same limitation.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-the-resolution-abandon-split-brain-dns-54)The resolution — abandon split-brain DNS

After all the above, the only reliable architecture was to stop trying to make one hostname behave two ways. The setup was changed to:

-   **`ha.yourdomain.com`** — proxied by Cloudflare, mTLS-enforced, the _only_ way in from outside. LAN clients never use this name.
-   **`lan.ha.yourdomain.com`** — DNS-only (grey-cloud) A/AAAA record pointing at the HA host's LAN IP, served by a local Nginx Proxy Manager instance with its own Let's Encrypt certificate. LAN clients use this name exclusively.

The Companion App supports both an External URL and an Internal URL, and it picks based on the connected wifi SSID — so it automatically uses the internal URL at home and the external URL away from home. The mTLS WAF rule protects only the external URL. The internal URL is protected by the boundary of the home network itself.

This is documented in **Part 8** as the recommended architecture. It sidesteps all of the DoH / IPv6 / split-brain issues entirely.

### [](https://community.home-assistant.io/t/guide-secure-remote-mtls-and-local-ha-access/1003197#p-3802461-lessons-55)Lessons

-   Split-brain DNS only works when **no client in the topology is using DoH**, which is no longer a realistic assumption.
-   Split-brain DNS on a **Cloudflare-fronted hostname** is especially brittle because Cloudflare always returns AAAA records, and those records always reach any client that can do encrypted DNS out-of-band.
-   When you hit a DNS-routing problem that only affects some apps or some browsers, check for DoH on every layer: the browser, the OS, the router, and any third-party secure-DNS extension.
-   If you need both external and internal access to a self-hosted service and the external path uses Cloudflare, **use two hostnames**. It is shorter, faster, more reliable, and easier to debug than any amount of split-brain cleverness.