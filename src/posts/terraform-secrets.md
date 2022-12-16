---
title: Using secrets stored in AWS Secrets Manager as environment variables for ECS container definitions. With Terraform.
description: Taking the managed approach to handling secrets for services using Terraform.
date: 2022-12-10
author: Christos Alexiou
tags:
  - terraform
  - security
---

Hello readers. I hope this piece of writing finds you healthy, sitting in a comfortable sofa or desk chair, and enjoying the wintry weather of your country.

Today, we are going to discuss environment variables in task definitions.

### The scenario

Given a managed cluster in AWS ECS, that was created using AWS Terraform provider, you are challenged to securely configure environment variables in the task definitions of each one of your cluster tasks, which you are also managing with Terraform - because Terraform all the things.

### The manifesto

There are countless ways of handling application secrets today, probably as many as the people who write software. I am not trying to preach on whether this is the right way to do it, or if it's the most secure or comfortable. But, I know for a fact that many people will end up in situations where this piece of writing will become a useful guide.

### The implementation

#### Directory structure

I was either lucky or farsighted when I was deciding on the structure of my Terraform code. This wonderful occasion allowed me to have the resources for the tasks, the services, and all of the paraphernalia in the same module letting me make direct references between them without worrying about inheritance or inclusion.

For this article, it is enough for the reader to be aware of the following directory structure:

```
├── services.tf
├── task_env_vars.tf
└── tasks. tf
├── definitions
│   └── template.json
```

#### Secrets in AWS Secrets Manager

AWS Secrets Manager helps you manage, retrieve, and rotate database credentials, API keys, and other secrets throughout their lifecycles. They encrypt the keys using the AWS Key Management Service. I suppose it is secure enough. If you are working for VISA or MasterCard and you think that the level of security they provide is not enough, you probably ought not to read this piece of writing.

I am not managing the secrets that are stored in AWS Secrets Manager with Terraform. I chose to manage those secrets using the AWS Console interface, and this is the path I've taken, although not ideal.

If the reader wants to go the route of managing the secrets with Terraform, the [required documentation is here](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/secretsmanager_secret) and it's extremely straightforward.

![The AWS Console interface for the AWS Secrets Manager service. Showcasing one created secret called "mailing-env".](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/t5o9yf7sqn3m6ioka61h.png)

For us, plebes, I am going to continue writing with the standpoint that the secrets are created through the interface, as I did, for instance, for the secret values that I want to be used as environment variables in a service that I wrote that's responsible for mailing.

##### Creating a secret

![The interface of AWS Console for creating a new secret in AWS Secrets Manager, with the option of "Other type of secret" selected.](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zci0okvwl7q8fhdb1nft.png)

When prompted by the wizard of the AWS Secrets Manager, we shall select the option **"Other type of secret"**. This will enable us to store secrets in a key/value structure, that will prove particularly useful when handling this data for use with our task definitions.

We can fill the text boxes with whatever we fancy.


![Filling one of the key value text boxes with key=X_API_KEY and value=123abc](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/75npoe1hy99q76pssvot.png)


Once we are done filling all the key/value pairs for the secret, we hit "Next" to finalize and name our secret, ignoring all the other options - at least for our case.

Since I am planning to use this secret in a service that does mailing, I named it _mailing-env_. 

Now, the secret is available to us through the AWS API and thus, the Terraform provider.

#### Retrieving the secret

With the secret available in AWS Secrets Manager, we can use the data source [`aws_secretsmanager_secret_version`](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/secretsmanager_secret_version) to retrieve the secret. 

```
data "aws_secretsmanager_secret_version" "mailing" {
  secret_id = "mailing-env"
}
```

#### Formatting the secret's values for further use

Because the received data are encoded and do not have the proper naming conventions for us to use right away in our task definitions, we have to do some local processing, to get them into an appropriate state.

```
locals {
  mailing_secrets = jsondecode(data.aws_secretsmanager_secret_version.mailing.secret_string)

  mailing_secrets_list = [
    for name, value in local.mailing_secrets : {
      name  = name
      value = value
    }
  ]
}
```

We are using the [`jsondecode`](https://developer.hashicorp.com/terraform/language/functions/jsondecode) function to get a representation of the result as Terraform language values.

The next endeavor is creating a list of environment variables in a way that we benefit from.

Since AWS expects the environment variables to be passed as [key/value pairs with `name` - `value` notation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/taskdef-envfiles.html), we are using terraform's `for` looping functionality to craft a list of objects.

#### Modelling the task definition with the help of templates

To make life easier and code cleaner, I've abstracted the task definition to a separate template file and stored it in `definitions/template.json`.

The bit that should be interesting to the reader of this essay is the following. The rest of the definition is boilerplate code that can be found in the [appropriate documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ecs_task_definition#example-using-container_definitions-and-inference_accelerator).

```
[
    {
        "environment": ${jsonencode(environment)}
    }
]

``` 

To use the template, we are relying on Terraform's [`templatefile`](https://developer.hashicorp.com/terraform/language/functions/templatefile) which renders the template for us and contains it in a single variable.

```
locals {
  mailing_definition = templatefile("${path.module}/definitions/template.json", {
    environment = local.mailing_secrets_list
  })
}
```

#### Using the task definition in a task resource

The last step, although not strictly relevant to this article, is taking advantage of the task definition we created.

Using [`aws_ecs_task_definition`](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ecs_task_definition) we can assign the value of the `local.mailing_definition` to the `container_definitions` attribute.

```
resource "aws_ecs_task_definition" "mailing" {
   container_definitions = local.mailing_definition
}
```

### The gist

The reason this article exists is that I didn't find a similar one. But, seldom is parthenogenesis in software, and this solution is no different.

These are the resources that led me to this solution.

https://stackoverflow.com/questions/67913171/how-to-pass-a-list-in-template-file-var-section-instead-of-string-in-terraform

https://github.com/hashicorp/terraform-provider-aws/issues/6503
