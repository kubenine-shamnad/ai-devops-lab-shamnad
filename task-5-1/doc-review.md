# AI Documentation Review — Task 3.41

I asked ChatGPT and Gemini to write a README for my Task 3.41 Terraform project.

## What my project really is

- Terraform `>= 1.5.0`, AWS provider `~> 6.0`
- Creates one S3 bucket only
- Region is `us-east-1`
- State uses a different S3 bucket and a DynamoDB lock table
- Tags: `common_tags` plus `Name` and `Purpose = "Terraform State Learning"`
- Outputs: bucket name, ARN, and region
- No versioning, encryption, or `force_destroy` on the test bucket

## Issues I found

- ChatGPT did not clearly say the state bucket and lock table must already exist
- Gemini said the backend was pre-existing, which was clearer
- Both skipped that the test bucket has no versioning, encryption, or `force_destroy`
- Both skipped that `terraform destroy` removes only the test bucket, not the state bucket or lock table
- ChatGPT said replace `ACCOUNT_ID` before use — fine for a sanitized example, but my real files already have the account id
- Gemini did not warn to keep `dynamodb_table` for Terraform 1.5–1.9 (those versions cannot use `use_lockfile`)
- Neither draft invented a fake resource — the problems were missing warnings and one unclear overview
