# QMF Cloud - Implementation Plan

> "QMF Cloud" is a temporary reference name of the product.

## Roadmap

> The roadmap is a top-level work breakdown ballpark estimate of the product development.

```mermaid
gantt
    title Work breakdown and task order
    dateFormat YYYY-MM-DD
    axisFormat %b-%d
    section Planning
        Define milestones: milestone, t, 2025-05-10,0d
        User's management: p14, 2025-05-05, 1w
        Backoffice: p15, after p14, 2w
        Agent deployment: p16, after p15, 1w
    section Documentation
        Define tools: milestone, 2025-05-06, 0d
    section Development
        Add users/passwords: de18, after p14, 2w
        Catalog search prod. ready: de21, after de18, 2w
        Backoffice v0.1: de22, after p15, 10w
        Agent run QMF objects: de23, after de21, 10w
        Service generate reports: de24, after de23, 10w
        Backoffice v0.2: de25, after de22, 10w
    section DevOps
        VPN: 2025-05-05, 3d
        DB/2 access: 2025-05-05, 3d
        GIT repository: 2025-05-05, 3d
        Cloud instances: 2025-05-05, 3d
        Build & deploy: do22, 2025-05-05, 3d
        Angent deployment: do23, after p16, 1w
        Backoffice deployment: do24, after de22, 1w
        Service deployment: do25, after do23, 1w
        Add Customer protocol: do26, after do24, 1w
    section Releases
        PoC: milestone, after do22, 0d
        Run Object as user: milestone, after de18, 0d
        Backoffice v0.1: milestone, after de22, 0d
        Preview v0.1: milestone, after de24, 0d
    section HR
        Product owner: milestone, 2025-05-25, 0d
        React/TS dev: milestone, 2025-05-12, 0d
        Java dev, Tech lead: milestone, 2025-05-12, 0d
```


## Components TODOs

### Agent
 - [ ] Connects to DB/2 with configured credentials, needs to use the credentials of the current user
 - [ ] Needs to be able to run QMF objects and provide result data
 - [ ] Probably more the one QMF repository per agent
 - [ ] Communication encryption
 - [ ] Confirm agent & service identity

### Service
 - [ ] Sync DB/2 catalog with the service using provided DB/2 credentials (TBD: service is per user? snapshot per user?)
 - [ ] Request QMF objects execution results and generate report previews

### Workbench (Frontend)
 - [ ] Send current user credentials to the service with each request/establish session
 - [ ] format the report locally?

### Backoffice
 - [ ] Manage "organizations" (QMF customers)
 - [ ] Manage users and their roles per organization by QMF Cloud admin (see below)
 - [ ] Instructions to install the agent
 - [ ] Manage regiterd agnents

## Questions

> They should be either removed or converted to TODOs

1. Will we have our own authentication system or will we use DB/2 authentication?
1. How associate db2 users with the services(snapshots) they can use? Dummy db/2 request?
1. Will the specific service (cluster) serve exactly one QMF Customer? (then security can be set by its config)
1. Should the service use user's credentials to sync catalog with the snapshot?
1. What is more than one user syncs the catalog at the same time? Should each user have its own snapshot?
1. Should be objects execution results be truncated to some limit to shorten the report preview?
1. Is it ok for service to be stateful? (e.g. remember the last viewed page of the report)

## Contexts
> this section needs to be confirmed

The final product is deployed across **three zones of responsibility**, each with distinct roles
and assets involved in the delivery and use of QMF Cloud.

## IBM

IBM is responsible for providing and maintaining the core infrastructure on the customer’s premises:

- **DB2 instance(s)** for data storage and access.
- **z/OS environment** to support:
    - QMF (Query Management Facility) running in TSO/CICS.
    - Installation and execution of the QMF Cloud Agent, a Java-based component running within the customer's environment.

## QMF Customer

The QMF Customer is the end-user organization and is represented by several internal roles:

### Roles:

- **QMF z/OS Admin**
    - Customer staff responsible for administering the z/OS environment.
    - Installs and upgrades the QMF Cloud Agent as needed.

- **QMF Cloud Admin**
    - Manages access and roles within the QMF Cloud environment.
    - Oversees the team of QMF Engineers, handling their permissions and configurations.

- **QMF Engineer**
    - Primary end-user of the QMF Workbench (frontend).
    - Develops and manages QMF objects using the QMF Workbench (frontend)
    - Allowed to run and preview reports in the development context, but **not supposed to execute real (production) reports**.

- **QMF User**
    - A general customer staff member who consumes production reports.
    - **Not to be contacted** for any operational tasks.
    - **Does not access the QMF Cloud** at any point.

## QMF Cloud

This is the hosted component of the product, provided by the QMF Cloud team. All parts of the solution not running on the customer's infrastructure fall under this responsibility.


### QMF Cloud Service(s)
 - Enhance catalog management with an indexed snapshot.
 - Generate report previews used during QMF object development.

### QMF Workbench (frontend)
A development tool used by QMF Engineers to:
 - Create and manage QMF objects.
 - Run and preview reports within the cloud context (non-production).

### QMF Cloud Agent
 - Deployed on the customer's z/OS system
 - Acts as the bridge between QMF Cloud and the on-premise DB2/QMF environment.

```mermaid
    C4Context
    title System Context for Production environment
    Enterprise_Boundary(customer, "QMF Customer") {
        Person(qmfUser, "QMF User", "A customer's staff to consume<br/>the reports.")
        Person(qmfZosAdmin, "QMF z/OS Admin", "A customer's staff to admin z/OS related task.")
        Person(qmfCloudAdmin, "QMF Cloud Admin", "A customer's staff to admin z/OS related task.")
        Person(qmfEngineer, "QMF Engineer", "A customer's staff to develop<br/>and manage QMF object.")
    }
    Enterprise_Boundary(imb, "IBM") {
        System_Boundary(zos, "z/OS Environment") {
            System(qmf, "QMF for TSO/CISC", "A runtime used by the end users.")
            SystemDb(db2, "DB/2 instance(s)", "A QMF objects catalog")
        }
        System_Boundary(javazos, "z/OS Java Environment") {
            System(agent, "Agent", "Manage QMF objects<br/>Run QMF objects")
        }
    }
    Enterprise_Boundary(cloud, "QMF Cloud") {
        System_Ext(backoffice, "QMF Backoffice", "Manage customers,<br/>users and permissions")
        System(frontend, "QMF Workbench<br/>(Frontend)", "QMF Engineer's<br/>tool to develop QMF objects")
        System(catalogService, "Catalog and<br/>reports(C&R) service", "Manage QMF catalog<br/>snapshot and generate reports")
    }

    Rel(qmfEngineer, frontend, "Search/create/modify/test<br/>QMF objects")
    Rel(qmfCloudAdmin, backoffice, "Manage cloud environment")
    BiRel(frontend, catalogService, "Search/modify QMF objects<br/>and request reports")
    BiRel(catalogService, agent, "JsonRPC over<br/>WebSocket API")
    Rel(qmfUser, qmf, "Request reports")
    Rel(qmfZosAdmin, agent, "Install/upgrade agent")
    Rel(agent, db2, "Read catalog<br/>and run QMF objects")
    Rel(catalogService, agent, "Request catalog snapshot<br/>and QMF objects execution")
    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="2")
```

During development, the product runs in a simplified environment, which is
primarily characterized by the absence of z/OS, as it is not required at
this stage

```mermaid
    C4Context
    title System Context for Dev environment
    Enterprise_Boundary("ds", "Data sources provider") {
        SystemDb(db2, "DB/2 instance(s)", "A sample QMF objects catalog")
    }
    Enterprise_Boundary("cloud", "Cloud provider") {
        System(agent, "Agent", "Manage QMF objects<br/>Run QMF objects")
        System(catalogService, "Catalog and<br/>reports(C&R) service", "Manage QMF catalog<br/>snapshot and generate reports")
        System(frontend, "QMF Workbench<br/>(Frontend)", "QMF Engineer's<br/>tool to develop QMF objects")
        System_Ext(backoffice, "QMF Backoffice", "Manage customers,<br/>users and permissions")
    }
    
Rel(agent, db2, "")
BiRel(catalogService, agent, "JsonRPC over<br/>WebSocket API")
BiRel(catalogService, frontend, "Search/modify QMF objects<br/>and request reports")
```

The development team generally consolidates communications internally and interacts with
stakeholders through the product owner and technical lead representatives. The product
owner is responsible for defining goals and reporting to stakeholders, while the technical
lead ensures that the technical implementation meets stakeholders’ requirements and expectations.

```mermaid
    C4Context
    title Development Communications

    Enterprise_Boundary(stakeHolders, "Stakeholders") {
        Person(representative, "Representative")
    }

    Enterprise_Boundary(devsTeam, "Developers (by roles)") {
        Person(powner, "Product owner", "Communications with s/h representative<br/>and project management")
        Person(tlead, "Teach lead", "Communications with s/h representative<br/>for technical issues<br/>code planning and review")
        Person(javaDev, "Java developer", "Backend developer")
        Person(reactDev, "React/TS developer", "Frontend developer")
        Person(tester, "Tester", "Continuous (auto)testing")
        Person(devOps, "DevOps engineer", "CI/CD")
        Person(security, "Security expert", "Communication and storage security")
    }

    BiRel(powner, representative, "")
    BiRel(tlead, representative, "")
    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

