# How to Create a Subdomain in Namecheap

## Step 1: Log in to Namecheap
1. Go to [Namecheap](https://www.namecheap.com/) and log in to your account.

## Step 2: Access Domain List
2. Click on **"Domain List"** from the left sidebar.
3. Find the domain for which you want to create a subdomain and click **"Manage"**.

## Step 3: Go to Advanced DNS Settings
4. Click the **"Advanced DNS"** tab at the top.

## Step 4: Add a New Subdomain Record
5. Under **"Host Records"**, click **"Add New Record"**.
6. Choose one of the following record types:
   
   - **A Record** → If you want to point to an IP address.
   - **CNAME Record** → If you want to point to another domain.
   
7. Fill in the details:
   - **Type:** Select `A Record` or `CNAME Record`.
   - **Host:** Enter your subdomain name (e.g., `app` for `app.yourdomain.com`).
   - **Value:**
     - For `A Record`, enter the destination IP address.
     - For `CNAME Record`, enter the target domain.
   - **TTL:** Set it to **Automatic** or **30 min**.

## Step 5: Save Changes
8. Click the **checkmark** (✔) to save the record.

## Step 6: Wait for DNS Propagation
- DNS changes can take **a few minutes to 48 hours** to fully propagate.