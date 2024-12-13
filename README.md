# A CI/CD pipeline to publish LaTeX documents

Everyone looking for a job prepares a resume but how many have thought of revolutionizing the way the resume is made and published? Most people use Word processors or the like to edit pre-built templates and share PDF copies with the employers. Have you ever seen anyone combine version control systems, CI/CD pipeline, cloud-based storage, domain/DNS management and caching to build and share resumes? Read on to find out how I make it possible! 

[Medium Article](https://blogs.nandinibhatt.me/a-ci-cd-pipeline-to-publish-latex-documents-d35d9eba9300)
```
https://blogs.nandinibhatt.me/a-ci-cd-pipeline-to-publish-latex-documents-d35d9eba9300
```

# Part 1 - Building a unique resume
- Used a free Overleaf template to build my resume
- Downloaded the LaTeX project files from Overleaf
- Used LaTeX workshop extension in VS code to edit the project locally

# Part 2 – Leveraging the power of AWS to host the resume 
- Used S3 to host the resources in a central location, ensured bucket allowed ACL policies and was public
- Made all the resources in the bucket public
- Modified DNS hosted zone on Route53 to create an alias (CNAME) of S3 bucket endpoint to point to our custom domain name
- This did not offer support for HTTPS based URLS because we can’t configure SSL certificate in this way
- Removed the newly inserted CNAME record
- Deployed a CloudFront Distribution in front of the S3 bucket hosting resources
- CloudFront offers very powerful caching across many edge locations throughout the world. It makes resources load in a zippy by serving them from edge locations
- Created an SSL certificate for the desired subdomain
- Created a subdomain CNAME entry in Route53 to point to the deployed CloudFront distribution endpoint

CAUTION: Whenever you update any resource in an S3 bucket backed by CloufFront, do not forget to invalidate the cache (by default, CloudFront caches files for 24 hours which is why you may not see the latest version if you do not explicitly invalidate the cache)

# Part 3 – Joining the dots 
- Pushed the repository to my GitHub account 
- Wrote a GitHub workflow to build a PDF from the repository
- We must use the latest version of Ubuntu provided by GitHub to compile the PDF. Previous versions of Ubuntu will install older versions of LaTeX that are not able to generate PDF on encountering undefined environment errors. However, the latest version of pdflatex works circles around this error and generates PDF.
- Install LaTeX and supplementary packages such as tex-live-fonts-extra (includes fontawesome)
- Build the PDF using pdflatex command and ensure that interaction flag is set to nonstop mode (to avoid hanging for user input in a CI/CD pipeline). This generates all the necessary intermediate files
- pdflatex throws an error but ignoring it does no harm to the final output. Therefore, we instruct GitHub to continue executing succeeding steps even if pdflatex throws an error
CAUTION: Each task must be performed in a separate step. As soon as any command in a step returns a non-zero exit code, all commands after it are skipped. However, commands in subsequent steps can still be executed.
- Build the references using biber without which the bibliography won’t be printed properly (.bcf generated using pdflatex is mandatory for this step to work). Then re-execute pdflatex command. (this step might not be needed depending on the environment and the references)
- Set up AWS access key and secret key for the GitHub repository
- AWS CLI comes pre-installed in the machines hosted by GitHub to run the CI/CD pipeline. We simply use AWS CLI to upload and make public the generated PDF to our AWS S3 bucket described above (if the resume is not made public with ACL, nobody will be able to access it)
- We then invalidate the cached version of the resume using AWS CLI (in order for the latest version to be served)
