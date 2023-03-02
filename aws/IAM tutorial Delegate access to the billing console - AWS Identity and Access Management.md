AWS account owners can delegate access to specific IAM users who need to view or manage the AWS Billing and Cost Management data for an AWS account. The instructions that follow will help you set up a pretested scenario. This scenario helps you gain hands-on experience configuring billing permissions without concern for affecting your main AWS production account. If you attach a managed policy to your IAM users instead of following this tutorial, you must first activate access to the AWS Billing and Cost Management console in [Step 1](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_billing.html?icmpid=docs_iam_console#tutorial-billing-step1).

This workflow has four basic steps.

![](https://docs.aws.amazon.com/images/IAM/latest/UserGuide/images/tutorial-billing.png)

**[Step 1: Activate access to billing data on your AWS test account](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_billing.html?icmpid=docs_iam_console#tutorial-billing-step1)**

If you create a single AWS account, only the AWS account owner ([AWS account root user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-user.html)) has access to view and manage billing information. IAM users cannot access billing data until the account owner activates IAM access and also attaches policies that provide billing actions to the user or role. To view additional tasks that require you to sign in as the root user, see [AWS Tasks that Require Account Root User](https://docs.aws.amazon.com/general/latest/gr/aws_tasks-that-require-root.html).

If you [create a member account](https://docs.aws.amazon.com/cli/latest/reference/organizations/create-account.html) using AWS Organizations, this feature is enabled by default.

**[Step 2: Create IAM policies that grant permissions to billing data](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_billing.html?icmpid=docs_iam_console#tutorial-billing-step2)**

After enabling billing access on your account, you must still explicitly grant access to billing data to specific IAM users or user groups. You grant this access with a customer managed policy.

**[Step 3: Attach billing policies to your user groups](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_billing.html?icmpid=docs_iam_console#tutorial-billing-step3)**

When you attach a policy to a user group, all members of that user group receive the complete set of access permissions that are associated with that policy. In this scenario, you attach the new billing policies to user groups containing only those users who require the billing access.

**[Step 4: Test access to the billing console](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_billing.html?icmpid=docs_iam_console#tutorial-billing-step4)**

After you've completed the core tasks, you're ready to test the policy. Testing ensures that the policy works the way you want it to.

## Prerequisites

Create a test AWS account to use with this tutorial. In this account create two test users and two test user groups as summarized in the following table. Be sure to assign a password to each user so that you can sign in later in Step 4.

| _Create user accounts_ | _Create and configure user group accounts_ |
| --- | --- |
| User name | User group name | Add user as a member |
| --- | --- | --- |
| FinanceManager | BillingFullAccessGroup | FinanceManager |
| FinanceUser | BillingViewAccessGroup | FinanceUser |

## Step 1: Activate access to billing data on your AWS test account

First, activate billing access for your test users in the AWS Billing and Cost Management console.

###### To activate IAM user and role access to the Billing and Cost Management console

1.  Sign in to the AWS Management Console with your root user credentials (specifically, the email address and password that you used to create your AWS account).
    
2.  On the navigation bar, choose your account name, and then choose [Account](https://console.aws.amazon.com/billing/home#/account).
    
3.  Next to **IAM User and Role Access to Billing Information**, choose **Edit**.
    
4.  Select the **Activate IAM Access** check box to activate access to the Billing and Cost Management console pages.
    
5.  Choose **Update**.
    

You can now use IAM policies to control which pages a user can access.

After you have activated IAM user access, you can attach IAM policies to grant or deny access to specific billing features. For more information about using policies to grant IAM users access to AWS Billing and Cost Management features, see [Using identity-based policies (IAM policies) for Billing and Cost Management](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/billing-permissions-ref.html) in the _AWS Billing User Guide_.

## Step 2: Create IAM policies that grant permissions to billing data

Next, create custom policies that grant both view and full access permissions to the pages within the Billing and Cost Management console. For general information about IAM permissions policies, see [Managed Policies and Inline Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html).

###### To create IAM policies that grant permissions to billing data

1.  Sign in to the AWS Management Console as a user with administrator credentials.
    
2.  Open the IAM console at [https://console.aws.amazon.com/iam/](https://console.aws.amazon.com/iam/).
    
3.  In the navigation pane, choose **Policies**, and then choose **Create policy**.
    
4.  On the **Visual editor** tab, choose **Choose a service** to get started. Then choose **Billing Console**.
    
5.  Follow these steps to create two policies:
    
    **Full access**
    
    1.  Select the check box next to **All Billing Console actions (aws-portal:\*)** under **Manual actions**. You do not need to select a resource or condition for this policy.
        
    2.  Choose **Next: Tags**.
        
    3.  Choose **Next: Review**.
        
    4.  On the **Review policy** page, next to **Name**, type `BillingFullAccess`, and then choose **Create policy** to save it.
        
    
    **Read-only access**
    
    1.  Repeat steps [3 and 4](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_billing.html?icmpid=docs_iam_console#tutorial-billing-repeat).
        
    2.  Choose **Select actions** and then select the check box next to **Read**. You do not need to select a resource or condition for this policy.
        
    3.  Choose **Next: Tags**.
        
    4.  Choose **Next: Review**.
        
    5.  On the **Review policy** page, for **Name**, type `BillingViewAccess`. Then choose **Create policy** to save it.
        
    
    To review descriptions for each of the permissions available in IAM policies that grant users access to the Billing and Cost Management console, see [Billing Permissions Descriptions](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/billing-permissions-ref.html#user-permissions).
    

## Step 3: Attach billing policies to your user groups

Now that you have custom billing policies available, you can attach them to their corresponding user groups that you created earlier. Although you can attach a policy directly to a user or role, we recommend (in accordance with IAM best practices) that you use user groups instead.

###### To attach billing policies to your user groups

1.  In the navigation pane, choose **Policies** to display the full list of policies available to your AWS account. To attach each policy to its appropriate user group, follow these steps:
    
    **Full access**
    
    1.  In the policy search box, type `BillingFullAccess`, and then select the check box next to the policy name.
        
    2.  Choose **Actions**, and then choose **Attach**.
        
    3.  In the identity (user, user group, and role) search box, type `BillingFullAccessGroup`, select the check box next to the name of the user group, and then choose **Attach policy**.
        
    
    **Read-only access**
    
    1.  In the policy search box, type `BillingViewAccess`, and then select the check box next to the policy name.
        
    2.  Choose **Actions**, and then choose **Attach**.
        
    3.  In the identity (user, user group, and role) search box, type `BillingViewAccessGroup`, select the check box next to the name of the user group, and then choose **Attach policy**.
        
    
2.  Sign out of the console, and then proceed to [Step 4: Test access to the billing console](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_billing.html?icmpid=docs_iam_console#tutorial-billing-step4).
    

## Step 4: Test access to the billing console

We recommend that you test access by signing in as each of the test users to learn what your users might experience. Use the following steps to sign in using both test accounts to see the difference between access rights.

###### To test billing access by signing in with both test users

1.  Use your AWS account ID or account alias, your IAM user name, and your password to sign in to the [IAM console](https://console.aws.amazon.com/iam).
    
    For your convenience, the AWS sign-in page uses a browser cookie to remember your IAM user name and account information. If you previously signed in as a different user, choose **Sign in to a different account** near the bottom of the page to return to the main sign-in page. From there, you can type your AWS account ID or account alias to be redirected to the IAM user sign-in page for your account.
    
2.  Sign in with each user using the steps provided below so you can compare the different user experiences.
    
    **Full access**
    
    1.  Sign in to your AWS account as the user FinanceManager.
        
    2.  On the navigation bar, choose **FinanceManager@`<account alias or ID number>`** , and then choose **Billing Dashboard**.
        
    3.  Browse through the pages and choose the various buttons to ensure that you have full modify permissions.
        
    
    **Read-only access**
    
    1.  Sign in to your AWS account as the user FinanceUser.
        
    2.  On the navigation bar, choose **FinanceUser@`<account alias or ID number>`**, and then choose **Billing Dashboard**.
        
    3.  Browse through the pages. Notice that you can display costs, reports, and billing data with no problems. However, if you choose an option to modify a value, you receive an **Access Denied** message. For example, on the **Preferences** page, choose any of the check boxes on the page, and then choose **Save preferences**. The console message informs you that you need **ModifyBilling** permissions to make changes to that page.
        
    

## Related resources

For related information found in the _AWS Billing User Guide_, see the following resources:

-   [Activating Access to the Billing and Cost Management Console](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/grantaccess.html#ControllingAccessWebsite-Activate)
    
-   [Allow full access to AWS services but deny IAM users access to the Billing and Cost Management console](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/billing-example-policies.html#ExampleAllowAllDenyBilling).
    
-   [Billing Permissions Descriptions](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/billing-permissions-ref.html#user-permissions)
    

For related information in the _IAM User Guide_, see the following resources:

-   [Managed policies and inline policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html)
    
-   [Controlling IAM users access to the AWS Management Console](https://docs.aws.amazon.com/IAM/latest/UserGuide/console_controlling-access.html)
    
-   [Attaching a policy to an IAM user group](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups_manage_attach-policy.html)
    

## Summary

You've now successfully completed all of the steps necessary to delegate user access to the Billing and Cost Management console. As a result, you've seen firsthand what your users billing console experience will be like. You can now proceed to implement this logic in your production environment at your convenience.