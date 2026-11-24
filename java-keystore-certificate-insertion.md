# Java Keystore Certificate Replacement and Application Deployment

## 1. Objective

To safely replace the existing `mydomain` SSL certificate in the Java Keystore (`cacerts`) with the new certificate (`STAR_mydomain_com.crt`) and deploy the updated application JAR files for Auth and Portal GW.

## 2. Prerequisites

- Root or appropriate administrative access to the server.
- Java installed at `/opt/jdk`.
- New SSL certificate available at `/tmp/devops/STAR_mydomain_com.crt`.
- Updated Auth and Portal GW JAR files available for deployment.
- Application deployment permissions.

**Environment Details**

| Parameter | Value |
|---|---|
| Java Keytool | `/opt/jdk/bin/keytool` |
| Java Keystore | `/opt/jdk/jre/lib/security/cacerts` |
| Certificate Alias | `mydomain` |
| New Certificate | `/tmp/devops/STAR_mydomain_com.crt` |
| Applications | Auth and Portal GW |

## 3. Procedure

### Step 1: Check Existing Certificate Alias

First, check whether the `mydomain` alias already exists in the Java Keystore.

```bash
/opt/jdk/bin/keytool -list \
-keystore /opt/jdk/jre/lib/security/cacerts \
-alias mydomain
```

Enter the keystore password when prompted.

**Expected outcomes:**

- If the alias exists, its certificate details will be displayed.
- If the alias does not exist, keytool will return an alias-not-found error.

### Step 2: Take a Backup of the Java Keystore

Before making any changes, take a backup of the existing `cacerts` file.

```bash
cp -p /opt/jdk/jre/lib/security/cacerts \
/opt/jdk/jre/lib/security/cacerts.bkp_$(date +%F_%H%M%S)
```

Verify the backup:

```bash
ls -lh /opt/jdk/jre/lib/security/cacerts*
```

**Purpose:** To ensure the original Java Keystore can be restored if the certificate replacement fails.

### Step 3: Delete the Existing Certificate

If the `mydomain` alias exists, delete the existing certificate.

```bash
/opt/jdk/bin/keytool -delete \
-alias mydomain \
-keystore /opt/jdk/jre/lib/security/cacerts
```

Enter the keystore password when prompted.

Verify that the alias has been deleted:

```bash
/opt/jdk/bin/keytool -list \
-keystore /opt/jdk/jre/lib/security/cacerts \
-alias mydomain
```

Expected result: Alias does not exist.

### Step 4: Import the New SSL Certificate

Import the new certificate into the Java Keystore using the same alias.

```bash
/opt/jdk/bin/keytool -importcert \
-file /tmp/devops/STAR_mydomain_com.crt \
-alias mydomain \
-keystore /opt/jdk/jre/lib/security/cacerts
```

Enter the keystore password when prompted.

When asked whether to trust this certificate, verify the certificate details and confirm with:

```text
yes
```

### Step 5: Verify the Newly Imported Certificate

Check the certificate details:

```bash
/opt/jdk/bin/keytool -list -v \
-keystore /opt/jdk/jre/lib/security/cacerts \
-alias mydomain
```

Verify the following:

- Alias name is `mydomain`.
- Certificate owner/Subject is correct.
- Issuer is correct.
- Validity dates are correct.
- SHA-256 fingerprint matches the approved new certificate.

### Step 6: Deploy the Updated Auth and Portal GW JAR Files

After successfully importing and verifying the new certificate, proceed with the application deployment.

**Applications to deploy:**

1. Auth
2. Portal GW

Deployment procedure:

1. Take a backup of the currently deployed Auth and Portal GW JAR files.
2. Stop the respective application services.
3. Replace the existing JAR files with the newly provided JAR files.
4. Verify the JAR file ownership and permissions.
5. Start the application services.
6. Check application startup logs for errors.

**Note:** Follow the existing application-specific deployment paths, service names, and startup procedures.

### Step 7: Post-Deployment Verification

Perform the following checks after deployment:

- Verify that both Auth and Portal GW services are running.
- Check application logs for SSL handshake or certificate-related errors.
- Verify that Auth and Portal GW endpoints are accessible.
- Confirm that the applications establish successful SSL/TLS connections.
- Confirm that no certificate trust-related errors appear in the logs.

## 4. Rollback Procedure

If certificate replacement or application deployment fails:

### Restore the Original Java Keystore

Identify the backup file created in Step 2 and restore it.

```bash
cp -p /opt/jdk/jre/lib/security/cacerts.bkp_YYYY-MM-DD_HHMMSS \
/opt/jdk/jre/lib/security/cacerts
```

Replace the backup filename with the actual backup file.

### Restore Previous Application JAR Files

1. Stop the affected application service.
2. Restore the previously backed-up JAR file.
3. Start the application service.
4. Verify service status and application logs.

## 5. Completion Criteria

The activity will be considered successful when:

- The old `mydomain` certificate has been replaced.
- The new certificate is successfully imported and verified.
- Updated Auth and Portal GW JAR files are deployed.
- Both application services are running successfully.
- No certificate-related errors are observed in application logs.
