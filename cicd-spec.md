v0.0.1

This specification describes the requirements for continuous software delivery (CI/CD) that includes automation, quality, and security as part of the delivery system (not an after thought).

This specification can be used as a guide to implement software delivery in your organization. You can create pipeline and repos that adhere to this specification.

# Specification

This specification describes 3 key components

- Application & Pipeline Scaffolding Template
- Application Verification Engine
- Team API

The template describes the requirements for the application repo and pipeline code. It includes reusable components that are common to many pipelines. It also includes the requirement to execute the rules containted in the verification engine.

The verification engine describes that automated verification

## Directives

The following derectives are key to the specification and detailed here so they can be refered to in the rest of the document.

### Portability Directive: Runtime must be independent of pipeline/build/deploy

Portability Directive: Runtime must be independent of pipeline/build/deploy tools as well as operating systems, such that it can run on almost any system with minimum configuration or expectations.

For example, if you’re using Jenkins as your build tool this would mean minimizing the usage of the jenkins pipeline syntax to avoid complexity running locally or on another build tool.

Implementation examples that would satisfy the required portability include:

- 3 Muskteers (eg: Make, Compose, Docker)
  - Containerization is the key for portability
  - Make or a similar agreed interface across the whole organization/userbase - that’s available on all likely build servers and workstations
- A Golang binary which can be easily installed on all systems

The portability directive also includes the kind of feedback the tools offers:

- When run without any parameters, output help which shows all methods/inputs/defaults for the tool
- When running a method, show expectations (such as inputs or environment arguments that are expected)
- If the method fails, show all missing arguments/vars not just the first failed one
- If the method fails, show the value for all key inputs/vars

## Application

This specification is focused on product teams producing software. There are some assumptions made about the nessasary platform, cloud, and infrastructure already available.

### Application - 'Deployable Unit' requirements

You application repo must follow the "Portability Directive - independent runtime".

You repo may include on or more deployable units. For every deployable unit you must produce an artifact with properties agreed within your development community (these will be used by the verification engine).

Artifact: [url to artifact with full name, eg https://nexus.com/group/foo:latest]

```
Repo:
Pipeline:
ProductID:
TeamID:
Type of development: [COTS, in-house, external]
Link to security report:
Link to test/quality report:
```

### Application & Pipeline Module Library

While some pipeline steps may be specific to your application, common steps should be shared among the organization and easily accessible.

The pipeline scaffolding (as well as development teams) should take advantage of modules in the library. If a module doesn’t exist it should be created. If it does exist it can be modified (either forked or updated) or another module can be created.

All modules should clearly define their inputs, env vars, purpose, category (build, test, secure), tools (sonar_v1,nexus_v2.3) they operate with. This way the organization can understand how modules are being used and where effort, help or collaboration might be useful. Module usage should also be captured.

Anyone can create a module at any time, this is the “trust”, “paved path”, “golden path” concept where it should be easier to use to extend a module, but if you need a new or different one it should be easy for you to create and use it - if it’s useful and popular it will be obvious over time, and worth the maintaince investment to create your own. 

Example modules:
- Run Sonaqube Quality Scan
- Push Artifact to Nexus
- Push Artifact to AWS ECR
- Run Blackduck scan
- Run Verification

## Application & Pipeline Scaffolding Template
The application scaffolding should automatically produce the repo/pipeline code which could be deployed to production, and pass an application verification for a given target language on it’s first commit.

For example, if your organization primarily uses Python and Golang for development, and Github, SonarQube, etc then the template could take the following inputs from the user:

```
New product name? foo
Type? Single container
What languge is this based on? Python/Golang/Scratch
Artifact store? Nexus
SourceControl? Github
Quality Tool? SonarQube, Custom, None
```

The environment and supporting systems need to exist, and the user would need to manually add necessary credentials after the one-off file generation happens. 

## Application Verification Engine
The verification engine runs against an artifact. It produces a report that verifies the artifact is ready for deployment.

### Assumptions
The engine must:

- Know which artifact store you’re using and be able to access it
Based on the type of store (ie know how to connect to a Nexus instance)
Have credentials for the specific instance of the store it needs to connect to
- Be able to access metadata associated with the artifact
  - In some cases within the artifact store
  - In other cases an external database
- Be able to access systems used to validate the artifact (eg Jira, SonarQube)

### Inputs

- Artifact Store Type - eg Nexus, Artifactory AWS ECR
- Artifact location - full url to artifact

### How to run

Portability Directive: Runtime must be independent of pipeline/build/deploy tools as well as operating systems, such that it can run on almost any system with minimum configuration or expectations.

For example, if you’re using Jenkins as your build tool this would mean minimizing the usage of the jenkins pipeline syntax to avoid complexity running locally or on another build tool.

Implementation examples that would satisfy the required portability include:

- 3 Muskteers (eg: Make, Compose, Docker)
  - Containerization is the key for portability
  - Make or a similar agreed interface across the whole organization/userbase - that’s available on all likely build servers and workstations
- A Golang binary which can be easily installed on all systems

The portability directive also includes the kind of feedback the tools offers:

- When run without any parameters, output help which shows all methods/inputs/defaults for the tool
- When running a method, show expectations (such as inputs or environment arguments that are expected)
- If the method fails, show all missing arguments/vars not just the first failed one
- If the method fails, show the value for all key inputs/vars

The rules engine must supply one single entrypoint to run all rules.

### Rules

The rules engine must be able to execute all the rules in it’s database in a single command. The simplest version of this maybe a folder for each rule contained in the same place as the engine itself. The rules engine must also allow running each rule independently. 

If using 3 musketeers here’s what it could look like:

```
# run all rules
cd repo && make validate

# run a specific rule
cd repo/rules/foo && make validate
```

Each rule must also follow the portability directive.

### Outputs

JSON or YAML payload:

```
Result: Pass/Fail
Total Number of Rules processed: n
Rules
  Rule Foo
  Result: Pass/Fail
  Inputs: X
  Why this rule exists: X
  What this rule does: X
  Message: X
  More Info: X
```

### Report Interface

A visual report (HTML, PDF, TXT or similar) should be generate from the JSON output, such that it can be output to the terminal (for txt) or saved and displayed later (for HTML/PDF/TXT).

## Team API

The Team API describes the type of information teams should make easily available. Teams may choose to add some of this information manually, while some might automate parts that change often. Some information makes more sense to stored as yml/json/README in the application-repo, while other information should be stored with the application-artifact.

Your organization should clearly specify the fields each team should make available, if the field is required, and example values.

What follows is an example template that your organization can use as a starting point.

### Team Level Info
<This should describe a single team - a group of individuals working together for a common goal. Ideally 1 team should be dedicated to 1 product, however the data capture allows for teams owning n-number of products. Even if your team doesn’t own a public-facing product, you should follow this model and consider any software or service you produce as being a product.

Stored in the main repo for the team as info-team.[md|yaml] - a team can only produce one of these. Alternative storage may be a single repo for team/product info to be stored across the org.
>

```
UniqueID for team:
Team created/start date: [yyyy-mm-dd]
Team name:
Team focus:
Team type: [product|platform]
Team org chart: [link to org chart, or image, or list of names and roles etc]
Team Ways of Working document:
Team Definition of Done:
```

### Product Level Info
<Products are always linked to one single team. Products may produce n-number deployable units of software.

Stored in the main repo for the product as info-product.[md|yaml]. Alternative storage may be a single repo for team/product info to be stored across the org.
>

```
UniqueID for product:
Product name:
Purpose of product:
Team that manages this product: [UniqueID for team]
Product Owner:
Product Type: [Crown Jewels|Other]
CustomerType: [Internal|External]
Customer: [Developers,Security, Internal Team X, All retail credit card customers]
Bug tracking URL: [Link to Jira board or JQL search or similar that shows all open bugs]
Architecture diagram:
```

### Repo Unit Level Info
<Every repo must have this one single file for it’s key information - if it doesn’t then it should not be treated as an a significant repo for the purposes of software delivery in the organization.

A repo may contain one or more deployable units (ie software that is built and then deployed. This could be a jar file, a docker image, etc) - or if it’s primary purpose is not to deploy software it may have no deployable units, however it may still play a role in software delivery (cloud, IaC, environment related automation etc).

A simplistic example would be one repo, producing one deployable unit. Other examples include a microservices architecture and/or monorepo where many (10-100) deployable units might be produced from a single commit.

Repos are ideally owned by a single team, or in some cases shared by several teams. Likewise repos should represent a single product, but in some cases may span both multiple products and teams.

Stored in the repo as info-repo.[md|yaml] - One per repo.

“*” indiciates that if the answers are the same across all repos for a given product or team they can be supplied at the product or team level. Can be overridden here.
>

```
TeamID: [x,y]
ProductID: [x,y]
Purpose: 
* Branching model:
Production Pipeline: <eg single url for the prod pipeline that deploys the units>
All other pipelines: [x,y,z] <eg urls for dev,stage,prod>
```

### Deployable Unit Level Info
<A deployable unit describes an artifact which can be deployed to production. Deployable units may share a common or related set of repos and pipelines & may be commonly deployed along side other deployable units, sharing similar characteristics.

Stored: Practically, this information would need to be automatically generated as part of the artifact publish process in the artifact metadata/properties.
>

Artifact: [url to artifact with full name, eg https://nexus.com/group/foo:latest]

```
Repo:
Pipeline:
ProductID:
TeamID:
Type of development: [COTS, in-house, external]
Link to security report:
Link to test/quality report:
```





