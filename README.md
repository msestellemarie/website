# www.estellecorey.com

## Initial Deploy

Install the East5th tool found [here](https://github.com/East5th/scripts).

Run `east5th aws login` and log in to the `estellecorey` AWS account.

Run `east5th aws create-stack` and select the `estellecorey` AWS account, set the stack name to `estellecorey` and select the `estellecorey.template` file for the template. This will deploy a CloudFormation stack that creates a Code Pipeline that triggers on a push to `main` in the `www.estellecorey.com` Github repo.

Then, approve the pending Github connection in AWS CodePipeline and add www records to Route 53 from Certificate Manager.

To test, push to `main` and validate that the CodePipeline successfully triggers.

If template changes are needed, update the template file and run `east5th aws update-stack`. Select the `estellecorey` AWS account, the `estellecorey` stack and select the `estellecorey.template` file for the template.
