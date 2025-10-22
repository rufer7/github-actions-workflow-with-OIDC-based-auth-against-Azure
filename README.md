<!--
  <<< Author notes: Header of the course >>>
  Read <https://skills.github.com/quickstart> for more information about how to build courses using this template.
  Include a 1280×640 image, course name in sentence case, and a concise description in emphasis.
  In your repository settings: enable template repository, add your 1280×640 social image, auto delete head branches.
  Next to "About", add description & tags; disable releases, packages, & environments.
  Add your open source license, GitHub uses Creative Commons Attribution 4.0 International.
-->

# GitHub Actions workflow with OIDC based authentication against Azure

_Create a workflow that uses GitHub Actions OIDC provider for authentication against Azure_

<!--
  <<< Author notes: Start of the course >>>
  Include start button, a note about Actions minutes,
  and tell the learner why they should take the course.
  Each step should be wrapped in <details>/<summary>, with an `id` set.
  The start <details> should have `open` as well.
  Do not use quotes on the <details> tag attributes.
-->

<!--step0-->

GitHub Actions workflows often need to access a cloud provider like AWS, Azure, GCP, ... for example for resource creation or for software deployment. Therefore usually long living secrets are used and stored as secrets in GitHub. GitHub Actions OpenID Connect (OIDC) provider allows you to configure your workflow to request a short-lived access token directly from the cloud provider. By updating your workflows to use OIDC tokens, you can adopt good security practices like no cloud secrets, authentication and authorization management as well as rotating credentials.

- **Who is this for**: Developers and DevOps engineers
- **What you'll learn**: Best practice authentication for workflows against Azure 
- **What you'll build**: A workflow that uses GitHub Actions OIDC provider for authentication against Azure
- **Prerequisites**: Azure tenant with a subscription
- **How long**: This course is 2 steps long and takes less than 15 minutes to complete.

## How to start this course

1. Above these instructions, click **Use this template**, right-click **Create a new repository** and click **Open link in new tab**.
   
   ![Use this template](https://user-images.githubusercontent.com/1221423/169618716-fb17528d-f332-4fc5-a11a-eaa23562665e.png)
3. In the new tab, follow the prompts to create a new repository.
   - For owner, choose your personal account or an organization to host the repository.
   - I recommend creating a public repository as private repositories will [use Actions minutes](https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions).
   ![Create a new repository](https://user-images.githubusercontent.com/1221423/169618722-406dc508-add4-4074-83f0-c7a7ad87f6f3.png)
4. After your new repository is created, wait about 20 seconds, then refresh the page. Follow the step-by-step instructions in the new repository's README.

<!--endstep0-->

<!--
  <<< Author notes: Step 1 >>>
  Choose 3-5 steps for your course.
  The first step is always the hardest, so pick something easy!
  Link to docs.github.com for further explanations.
  Encourage users to open new tabs for steps!
  TBD-step-1-notes.
-->

<details id=1 open>
<summary><h2>Step 1: Configure OIDC in Azure</h2></summary>

_Welcome to "GitHub Actions workflow with OIDC based authentication against Azure"! :wave:_

In this step you'll configure OIDC in your Azure tenant.

**What is _Azure_**: Azure is the cloud platform of Microsoft.

**What is a _tenant_**: A Microsoft Entra ID tenant is a reserved Microsoft Entra ID service instance that an organization receives and owns once it signs up for a Microsoft cloud service such as Azure, Microsoft Intune, or Microsoft 365.

**What is _OIDC_**: OpenID Connect or OIDC is an identity protocol that utilizes the authorization and authentication mechanisms of OAuth 2.0.

**What is a branch?**: A [branch](https://docs.github.com/en/get-started/quickstart/github-glossary#branch) is a parallel version of your repository. By default, your repository has one branch named `main` and it is considered to be the definitive branch. You can create additional branches off of `main` in your repository. You can use branches to have different versions of a project at one time.

### :keyboard: Activity: Configure OIDC in Azure

1. Open a new browser tab, and work on the steps in your second tab while you read the instructions in this tab
1. [Create a Microsoft Entra application with a service principal](https://learn.microsoft.com/en-us/azure/developer/github/connect-from-azure-openid-connect?WT.mc_id=MVP_344197#prerequisites) by following the steps under Option 1
    - Assign role `Contributor` with scope subscription (on subscription level) to the application (for a detailed step-by-step manual, see [here](https://learn.microsoft.com/en-us/azure/role-based-access-control/role-assignments-portal?tabs=delegate-condition&WT.mc_id=MVP_344197#step-1-identify-the-needed-scope))
1. [Add federated credentials](https://learn.microsoft.com/en-us/entra/workload-id/workload-identity-federation-create-trust?pivots=identity-wif-apps-methods-azp&WT.mc_id=MVP_344197#github-actions) by following the steps under the link
    - `Entity type`: `Branch`
    - GitHub branch name: `deploy-resource-group`
1. [Create GitHub secrets](https://learn.microsoft.com/en-us/azure/developer/github/connect-from-azure-openid-connect?WT.mc_id=MVP_344197#create-github-secrets) by following the steps under the link
1. If you are done, create a new branch with name `deploy-resource-group`
1. Wait about 20 seconds then refresh this page for the next step

</details>

<!-- 
  <<< Author notes: Step 2 >>>
  Start this step by acknowledging the previous step.
  Define terms and link to docs.github.com.
-->

<details id=2>
<summary><h2>Step 2: Create workflow</h2></summary>

_You did configure OIDC in Azure and created a branch! :tada:_

Configuring OIDC in Azure allows you to authenticate in a GitHub Actions workflow without the need of storing an access token in GitHub!

**What is a _workflow_**: A workflow is a configurable automated process that will run one or more jobs. Workflows are defined by a YAML file checked in to your repository and will run when triggered by an event in your repository, or they can be triggered manually, or at a defined schedule.

### :keyboard: Activity: Create workflow

The following steps will guide you through the process of creating a GitHub Actions workflow.

1. On the **Code** tab, make sure you're on your new branch `deploy-resource-group`
1. Click on tab `Settings`
1. In section `Default branch` switch the default branch to `deploy-resource-group` (click on button <--> to switch default branch)
1. Click on tab `Actions`
1. Click on button `new workflow`
1. Choose `Simple workflow` and click `Configure`
1. Rename file to `my-first-workflow.yml`
1. Replace content of `.yml` file with the following content
```yml
name: Run Azure Login with OpenID Connect and PowerShell
on: [push]

permissions:
      id-token: write
      contents: read
      
jobs: 
  Windows-latest:
      runs-on: windows-latest
      steps:
        - name: OIDC Login to Azure Public Cloud with AzPowershell (enableAzPSSession true)
          uses: azure/login@v1
          with:
            client-id: ${{ secrets.AZURE_CLIENT_ID }}
            tenant-id: ${{ secrets.AZURE_TENANT_ID }}
            subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }} 
            enable-AzPSSession: true

        - name: 'Create resource group with PowerShell action'
          uses: azure/powershell@v1
          with:
             inlineScript: |
               New-AzResourceGroup -Name MyFirstResourceGroup -Location "South Central US"
             azPSVersion: "latest"
```
10. Click `Commit changes...` button
11. Wait until the GitHub Actions workflow finished and then refresh this page for the next step

</details>

<!--
  <<< Author notes: Finish >>>
  Review what we learned, ask for feedback, provide next steps.
-->

<details id=X>
<summary><h2>Finish</h2></summary>

_Congratulations friend, you've completed this course!_

<img src=https://github.com/rufer7/github-actions-workflow-with-OIDC-based-auth-with-Azure/blob/main/fireworks.jpg alt=celebrate width=300 align=right>

Here's a recap of all the tasks you've accomplished in this course:

- You configured OIDC in Azure by adding the Federated Credentials to Azure
- You created your first GitHub Actions workflow that uses GitHub Actions OIDC provider for authentication against Azure
- The execution of this workflow create a resource group in your Azure tenant by using best practice authentication

### What's next?

- [I'd love to hear what you thought of this course](mailto:m.rufer@gmx.ch)
- [Check out using environments for deployment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)
- [Check out reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [Check out Assigning permissions to jobs](https://docs.github.com/en/actions/using-jobs/assigning-permissions-to-jobs)
- To find projects to contribute to, check out [GitHub Explore](https://github.com/explore)

</details>

<!--
  <<< Author notes: Footer >>>
  Add a link to get support, GitHub status page, code of conduct, license link.
-->

---

Get help: [About security hardening with OpenID Connect]([TBD-support-link](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)) &bull; [Review the GitHub status page](https://www.githubstatus.com/)

&copy; 2025 Marc Rufer &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [License](https://github.com/rufer7/github-actions-workflow-with-OIDC-based-auth-with-Azure/blob/main/LICENSE)
