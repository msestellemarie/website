# www.estellecorey.com

## Initial Deploy

Run `yarn deploy-estellecorey --profile ${AWS_PROFILE} --action create` from the base of the project. This will deploy a CloudFormation stack that creates a Code Pipeline that triggers on a push to `main` in the `www.estellecorey.com` Github repo.

Then, approve the pending Github connection in AWS CodePipeline in the associated AWS account. Also, add www records to Route 53 from Certificate Manager.

To test, push to `main` and validate that the CodePipeline successfully triggers.

If template changes are needed, update the template file and run `yarn deploy-estellecorey --profile ${AWS_PROFILE} --action update`.
