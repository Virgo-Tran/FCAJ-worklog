---
title : "CI/CD pipeline"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.2.3 </b> "
---

The continuous integration and continuous deployment pipeline runs **independently of the main business flow** and is triggered whenever the source code changes:

1. **GitHub** stores the backend service source code; every commit or merge triggers the pipeline.
2. **AWS CodeBuild** compiles the source, runs the tests, and packages the result into a Docker image.
3. **Amazon ECR** stores each built image, versioned by tag.
4. **AWS CodeDeploy** pulls the newest image from ECR and performs a rolling deployment onto the ECS Cluster with no service interruption.

```text
GitHub  →  CodeBuild  →  ECR  →  CodeDeploy  →  ECS Cluster
 commit     build+test    image    rolling        Fargate tasks
                          (tag)    deployment
```

{{% notice note %}}
This model lets the development team ship new versions quickly and safely, with the ability to roll back when needed — in keeping with modern DevOps practice.
{{% /notice %}}
