# Advanced-configuration-management-with-Kustomize-and-AWS
here i will be submitting project on the above subject
## The steps i took
I started by preparing my Kubernetes repository so that GitHub Actions and Kustomize would recognize everything without ambiguity. Inside my project root, I confirmed that three directories existed with precise names and locations: .github/workflows, base, and overlays. The .github/workflows directory is the standard location GitHub Actions scans for workflow definitions; base is where I keep the shared, canonical Kubernetes manifests; and overlays is the place for environment-specific adjustments that layer on top of the base. Where any of these folders were missing, I created them exactly as spelled to avoid path or case-sensitivity issues.

After setting up the folder structure, I created the workflow file that would orchestrate my automation. Within .github/workflows, I added a new file called main.yml. I double-checked the full path (.github/workflows/main.yml) and verified that the .github directory name began with a leading dot and that the file extension was .yml, not .yaml or anything else, to be consistent with the instructions.

Next, I opened .github/workflows/main.yml and configured the workflow’s identity and trigger. I assigned the workflow the name “Deploy with Kustomize” so it would be easy to recognize in the GitHub Actions interface. Then I defined the trigger so that any push to the main branch would launch this workflow. In other words, each time I pushed changes to main, the automation would run without me needing to do anything manually.

With the trigger ready, I declared the job that the workflow would execute. I created a job called deploy, specified that it should run on ubuntu-latest, and set up an initially empty steps list that I would populate with the specific actions. Choosing ubuntu-latest ensures a consistent Linux environment with broad tool support and predictable behavior across runs.

I then populated the steps. The very first step I added was a repository checkout so the runner would have the current version of my codebase. I used the standard GitHub action actions/checkout@v2 to pull the repository contents into the runner’s workspace, guaranteeing that subsequent steps could read my manifests and Kustomize files.

After checkout, I installed the two essential tools required by later stages: kubectl and kustomize. I added a step that uses azure/setup-kubectl@v1 to install kubectl, the command-line client for interacting with Kubernetes clusters. Immediately after, I added a step that uses imranismail/setup-kustomize@v1 to install kustomize, the tool that composes my base manifests with overlays and generators into the final, environment-ready resources.

At this point, I recorded an important requirement shown in the materials: when I apply Kustomize configurations from CI/CD, my GitHub Actions runner must be able to reach and authenticate to the target Kubernetes cluster. For this stage of the project, I only acknowledged that connectivity is required; I did not add extra steps beyond what was shown.

To validate the workflow, I performed a controlled configuration change that would trigger a pipeline run. I opened one of my overlay manifests where a Deployment’s replica count is defined, changed the replica number deliberately, and saved the file. Then, from the project root in my terminal, I ran three commands in order. First, I staged the modifications with git add .. Second, I committed the update with the message “Update Kustomize configuration”. Third, I pushed to the remote main branch with git push origin main. Because my workflow is wired to run on pushes to main, this push automatically started the pipeline.

I monitored the automation through the GitHub web interface. I navigated to the repository’s Actions tab, located the run labeled “Deploy with Kustomize,” opened the run details, and examined the individual steps and their logs. I confirmed that the workflow executed as expected. Per the project’s scope, I stopped my validation at reviewing the run and logs since the instructions did not require any extra verification beyond that point.

I also captured the optional notes shown about environment variables and triggers. I documented that sensitive data—such as cluster credentials—should be stored as environment variables or GitHub Actions secrets rather than embedded in manifests or workflow files. I additionally noted that workflows can be configured to trigger on pull requests, path filters, or tags; however, I did not implement any of these optional trigger variations because the project only required acknowledging them.

I reviewed the guidance on structuring complex configurations. I documented that my repository should scale to multiple applications or services and that a hierarchical layout—organized by application and then by environment—keeps things tidy. I reaffirmed the base-and-overlays model: place shared, reusable resources in base, and put environment-specific variations in overlays, so changes remain targeted and controlled.

Next, I implemented the caching optimization to speed up CI runs. Inside .github/workflows/main.yml, within the deploy job’s steps and before any build or dependency-heavy actions, I added a cache step that uses actions/cache@v2 with the exact inputs shown. I set path to /tmp/.buildx-cache to define where cached artifacts should be stored. I set key to ${{ runner.os }}-buildx-${{ github.sha }} so each run uses a unique cache key tied to the operating system and the current commit SHA. I also set restore-keys to the prefix ${{ runner.os }}-buildx- so that if the exact key is unavailable, a compatible cache sharing that prefix can still be restored. With this in place, subsequent runs can reuse cached build layers or dependencies, which reduces execution time and improves consistency. In the Actions logs, I can see whether a cache was restored or created, which confirms the caching behavior.

I wrote down the practices for optimizing Kustomize configurations that were listed. First, I will split large Kustomize setups into smaller, more manageable parts, so each unit is easier to maintain. Second, I will continue to leverage base and overlays effectively, reusing common definitions and isolating environment differences. Third, I will periodically review and refactor the configuration to eliminate duplication, clarify intent, and improve reliability.

To avoid common pitfalls, I adopted the approach of using Kustomize generators for dynamic values. I added a ConfigMap generator in kustomization.yaml. In that file, I declared a configMapGenerator entry named my-app-config with two literals: app_name=MyKustomizeApp and log_level=debug. This instructs Kustomize to produce a ConfigMap named my-app-config containing those key-value pairs.

I then referenced that ConfigMap in my Deployment. In deployment.yaml (apiVersion: apps/v1, kind: Deployment), I verified that my container is named app-container and uses image myapp:latest, as indicated. Within that container’s specification, I inserted envFrom with a configMapRef pointing to name: my-app-config. This wiring causes the keys defined in the ConfigMap to be injected as environment variables in the running container.

I also added a Secret generator in kustomization.yaml to handle sensitive configuration. In that file, I declared a secretGenerator entry named my-app-secret with two literals: username=admin and password=VGhp c0lzU2VjcmV0IQ==. I noted the instruction that literals for secrets should be base64-encoded or sourced from files for stronger security practices. When rendered, this produces a Secret named my-app-secret.

I connected the Secret to the Deployment as well. In deployment.yaml, inside the same container configuration, I added envFrom with secretRef pointing to name: my-app-secret. This allows the container to receive sensitive settings as environment variables without exposing them directly in the manifest.

I recorded the clarification that when I apply my Kustomize build, the generators are realized into actual Kubernetes resources—specifically, a ConfigMap and a Secret—at apply time. This separation between resource definitions and configuration values improves maintainability and security, since I keep values out of the base manifests and let Kustomize materialize them as needed.

I documented the change-management practices listed. Before configuration changes are applied, I will ensure they go through a review process. I will use GitOps patterns so changes are version-controlled and reviewed via pull requests, and I will test updates in a staging environment before promoting them to production to minimize risk.

I also committed to leveraging community resources. I will participate in discussions and ask questions on platforms such as Stack Overflow and Kubernetes Slack so I can learn common resolutions, best practices, and evolving patterns from the wider community.

For ongoing improvement, I committed to continuous learning by consulting the official Kustomize documentation on a regular basis. This helps me keep up with feature updates, recommended patterns, and changes that might influence how I structure or apply my configurations.

Finally, I documented the AWS-specific guidance exactly as presented. For deployment, I can use Amazon EKS to obtain a managed Kubernetes environment. I can provision EKS using eksctl or through the AWS Management Console. I must ensure that the AWS CLI is correctly configured to access the EKS cluster. For CI/CD within AWS, I can employ AWS CodePipeline to create a streamlined pipeline, configure CodePipeline to invoke Kustomize during deployment stages, and, where necessary, integrate other AWS services such as CodeCommit, CodeBuild, and CodeDeploy. I ended my work here because this is precisely where the provided instructions concluded.

## Feedback Request
I adhered strictly to the scope of the provided materials, followed the steps in sequence from initial repository preparation through pipeline execution and best practices, and stopped where the instructions ended. To strengthen my understanding without changing the required deliverables, I focused on the reasoning behind each configuration choice and verified file paths, names, and keys meticulously. As additional learning outside the formal scope, I briefly reviewed relevant sections of the Kustomize documentation to confirm how configMapGenerator and secretGenerator are rendered, and I skimmed community discussions related to actions/cache usage and overlay organization. Any exploratory testing I performed was done in a separate scratch space and not committed to the project repository.

## Conclusion
This report recounts everything I completed: establishing the correct repository structure; creating .github/workflows/main.yml; defining the workflow name and push-to-main trigger; declaring the deploy job on ubuntu-latest; adding the checkout step; installing kubectl and kustomize; acknowledging the requirement for cluster connectivity from the CI runner; making a small overlay change; staging, committing, and pushing to main; and inspecting the workflow run in the Actions tab. I captured the optional notes on environment variables and enhanced triggers without implementing extras. I documented the organizational guidance for complex configurations, implemented the caching step with the exact path, key, and restore-keys values, and wrote down the optimization practices for Kustomize. I created and referenced a ConfigMap via configMapGenerator and a Secret via secretGenerator, reflected the clarification about generator-produced resources at apply time, and recorded disciplined update and review processes alongside community and continuous-learning commitments. I also documented the AWS notes for EKS setup and CodePipeline integration exactly as listed. The images below depict these steps in sequence.

![1img](./1img)
![2img](./2img)
![3img](./3img)
![4img](./4img)
![5img](./5img)
![6img](./6img)
![7img](./7img)
![8img](./8img)
![9img](./9img)
![10img](./10img)
![11img](./11img)
![12img](./12img)
![13img](./13img)
![14img](./14img)
![15img](./15img)
![16img](./16img)

