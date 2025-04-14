# Hosting a Static Website on S3 with HTTPS Using CloudFront (April 2025)

### Step 1: Create an S3 Bucket

- Go to the **S3 Console** and create a new bucket with a globally unique name, e.g., `my-static-site-04-2025`.
- Leave **Block all public access** checked (default setting).
- Click **Create Bucket**.

### Step 2: Upload Website Files

- Upload your `index.html` and `error.html` files to the bucket.

### Step 3: Enable Static Website Hosting

- Go to the **Properties** tab of your bucket.
- Scroll down to **Static website hosting**.
- Enable it.
- Set the index document to `index.html`.
- Optionally, set the error document to `error.html`.

### Step 4: Update Bucket Policy

To allow public read access for website hosting, update your bucket policy like this:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-static-site-2025/*"
    }
  ]
}
```

> **Note:** At this stage, the website is publicly accessible but only over HTTP, not HTTPS. There's also higher latency due to lack of CDN.

---

## Step 5: Distribute via CloudFront (for HTTPS and Low Latency)

### Create a CloudFront Distribution

- Go to the **CloudFront Console**, click **Create Distribution**.

**Origin settings:**
- **Origin domain**: Choose your S3 bucket (will look like `my-static-site-04-2025.s3.<region>.amazonaws.com`).
- **Origin access**: Choose **Origin access control settings (recommended)**.
- Click **Create new OAC**, select:
  - **Signing behavior**: `Sign requests (recommended)`
  - Then click **Create**.

**Default behavior settings:**
- **Viewer protocol policy**: `Redirect HTTP to HTTPS`
- **Allowed HTTP methods**: `GET, HEAD`
- **Default root object**: `index.html`

Click **Create Distribution**.

### Step 6: Restrict S3 Bucket Access to CloudFront

CloudFront will generate a bucket policy. Paste this into your S3 bucket policy to ensure only CloudFront can access your content (replace the placeholder with your actual CloudFront OAC info).

After this, remove the earlier public access policy to make the bucket private again.

---

### Done!  

