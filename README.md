# Static Site — GitHub → Jenkins → AWS S3

A simple HTML/CSS site that is continuously deployed to an AWS S3 static
website by a Jenkins pipeline whenever changes are pushed to GitHub.

## Structure

```
.
├── index.html        # Home page
├── error.html        # 404 page (served by S3 for missing keys)
├── css/styles.css    # Styles
├── Jenkinsfile       # Declarative Jenkins pipeline (checkout → deploy to S3)
└── README.md
```

## How it works

1. Push to the `main` branch on GitHub.
2. Jenkins (polling SCM or a GitHub webhook) triggers the pipeline.
3. The pipeline runs `aws s3 sync` to upload the site to the S3 bucket.
4. The site is served from the S3 website endpoint.

## AWS target

| Setting | Value |
| --- | --- |
| Bucket | `jenkins-static-site-799183605681` |
| Region | `ap-south-1` |
| Website endpoint | `http://jenkins-static-site-799183605681.s3-website.ap-south-1.amazonaws.com` |

## Jenkins setup

1. **AWS access** — The Jenkins EC2 instance has the IAM role
   `JenkinsS3DeployRole` (with `AmazonS3FullAccess`) attached, so the AWS CLI
   authenticates automatically — no access keys are stored in Jenkins.

2. **Prerequisites on the agent** — the AWS CLI v2 must be installed and on
   `PATH`.

3. **Pipeline job** — Create a *Pipeline* (or *Multibranch Pipeline*) job,
   point it at this GitHub repo, and set *Pipeline script from SCM* →
   `Jenkinsfile`.

4. **Trigger** — Enable *GitHub hook trigger for GITScm polling* (with a
   GitHub webhook) or *Poll SCM*.

## Local preview

Open `index.html` in a browser, or serve the folder:

```bash
python -m http.server 8000
```
