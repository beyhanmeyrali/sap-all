### Resources for Each Chapter

I've compiled a list of relevant links from official SAP sources (primarily help.sap.com and blogs/community.sap.com) based on targeted web searches for each chapter topic. These are tailored for old ABAP and Fiori developers transitioning to BTP, RISE, and Joule. I selected the top 3–5 most relevant results per chapter (filtering out less direct matches). Each includes the title, URL, and a brief snippet for context. An AI agent can browse these URLs (e.g., via browse_page tool) to extract detailed content and generate the chapter markdown files.

For the **Preface** (general intro for ABAP devs), I used overlapping results from Chapter 1 searches.

#### Preface: Who This Book Is For (Classic ABAP & Fiori Developers)
- [SAP BTP ABAP Environment - Frequently Asked Questions](https://community.sap.com/t5/technology-blog-posts-by-sap/sap-btp-abap-environment-frequently-asked-questions/ba-p/14174603) – Overview for ABAP devs on cloud-optimized ABAP platform.
- [Learn how to develop in ABAP using SAP BTP ABAP Environment](https://community.sap.com/t5/technology-blog-posts-by-sap/learn-how-to-develop-in-abap-using-sap-btp-abap-environment/ba-p/13492682) – Tutorial-style intro to ABAP in BTP for traditional developers.
- [SAP BTP Developer's Guide](https://help.sap.com/docs/btp/btp-developers-guide/btp-developers-guide) – High-level guide for developers starting with BTP.

#### Chapter 1: What Even Is SAP BTP?
- [SAP BTP ABAP Environment](https://pages.community.sap.com/topics/btp-abap-environment) – PaaS offering for ABAP development in cloud.
- [SAP BTP ABAP Environment - Frequently Asked Questions](https://community.sap.com/t5/technology-blog-posts-by-sap/sap-btp-abap-environment-frequently-asked-questions/ba-p/14174603) – Explains BTP as cloud ABAP platform.
- [SAP BTP Developer's Guide](https://help.sap.com/docs/btp/btp-developers-guide/btp-developers-guide) – Overview of BTP as integrated cloud platform.
- [Learn how to develop in ABAP using SAP BTP ABAP Environment](https://community.sap.com/t5/technology-blog-posts-by-sap/learn-how-to-develop-in-abap-using-sap-btp-abap-environment/ba-p/13492682) – Intro to BTP for ABAP know-how.
- [SAP Business Technology Platform](https://help.sap.com/docs/btp/sap-business-technology-platform/sap-business-technology-platform) – Core definition and portfolios.

#### Chapter 2: The BTP Architecture – Your New Apartment Building
- [Account Model](https://help.sap.com/docs/btp/sap-business-technology-platform/account-model) – Explains global accounts, subaccounts, entitlements.
- [Basic Platform Concepts](https://help.sap.com/docs/btp/btp-admin-guide/basic-platform-concepts) – Diagram of global account, directories, subaccounts, regions, entitlements.
- [Entitlements and Quotas](https://help.sap.com/docs/btp/sap-business-technology-platform/entitlements-and-quotas) – How entitlements are managed at global and subaccount levels.
- [Configure Entitlements and Quotas for Subaccounts](https://help.sap.com/docs/btp/sap-business-technology-platform/configure-entitlements-and-quotas-for-subaccounts) – Practical distribution to subaccounts.
- [Working with Global Accounts, Directories, and Subaccounts Using the btp CLI](https://help.sap.com/docs/btp/sap-business-technology-platform/working-with-global-accounts-directories-and-subaccounts-using-btp-cli) – CLI management for architecture elements.

#### Chapter 3: RISE with SAP Demystified
- [Simplification List for SAP S/4HANA 2021](https://help.sap.com/doc/f2591a6901344c97a5e2029cc8f3703e/2021/en-US/SIMPL_OP2021.pdf) – Mentions RISE in context of S/4HANA editions (limited but relevant).
- [Quality Management (QM)](https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/9905622a5c1f49ba84e9076fc83a9c2c/e2f8f94be737403696eeea0e2be80d87.html) – Ties into RISE's process improvements.

(Note: Searches returned fewer direct "demystified" matches; these cover S/4HANA editions in RISE. Suggest browsing these for deeper extraction.)

#### Chapter 4: How RISE and BTP Fit Together
- [Development](https://help.sap.com/docs/btp/sap-business-technology-platform/btp-development) – How BTP environments support development in RISE contexts.
- [Step-by-Step Guide for Process-Driven Transformation with SAP's Integrated Toolchain](https://help.sap.com/docs/leanix/ea/step-by-step-guide-for-rise-transformation-with-integrated-toolchain) – RISE journey with BTP tools.
- [Use SAP Build Process Automation](https://help.sap.com/docs/build-process-automation/sap-build-process-automation/using-sap-build-process-automation) – Integration of BTP services in RISE.
- [Integrating Joule with SAP Solutions](https://help.sap.com/doc/de3af3c0f81642dbaa4d36172ed57a72/CLOUD/en-US/79bfc83ab386450c8cd9c7937ce26a3a.pdf) – Example of BTP (Joule) in RISE setups.
- [Activate Your SAP Build Process Automation Package](https://help.sap.com/docs/build-process-automation/sap-build-process-automation/activate-sap-build-process-automation-starter-pack) – BTP access in RISE global accounts.

#### Chapter 5: Destinations – The Phone Book for Connections
- [Creating a Destination to the ABAP System for SAP Business Application Studio](https://help.sap.com/docs/btp/sap-business-technology-platform/creating-destination-to-abap-system-for-sap-business-application-studio) – Setup tutorial.
- [Creating a Destination](https://help.sap.com/docs/btp/sap-business-technology-platform/create-destination) – Basic creation steps.
- [Create a Destination](https://help.sap.com/docs/sap-btp-abap-environment/abap-environment/create-destination?q=Transport&version=Cloud) – In ABAP context.
- [SAP BTP: Creating and Configuring a Destination](https://help.sap.com/docs/excise-tax-management/administration/sap-btp-creating-and-configuring-destination) – Configuration with Cloud Connector.
- [Full-Stack ABAP Project Creation Using Development Tools](https://help.sap.com/docs/build-service/build-service-guide/create-full-stack-abap-project) – Using destinations in projects.

#### Chapter 6: ABAP in the Cloud – The ABAP Environment
- [ABAP RESTful Application Programming Model](https://help.sap.com/docs/ABAP_PLATFORM_NEW/fc4c71aa50014fd1b43721701471913d/289477a81eec4d4e84c0302fb6835035.html) – RAP guide for ABAP Cloud.
- [ABAP RESTful Application Programming Model](https://help.sap.com/docs/btp/sap-business-technology-platform/abap-restful-application-programming-model) – Overview of RAP in BTP.
- [Tutorials for ABAP Cloud](https://help.sap.com/docs/btp/btp-developers-guide/tutorials-for-abap-cloud) – Step-by-step for RAP and ABAP Cloud.
- [Tutorial Overview](https://help.sap.com/docs/sap-btp-abap-environment/abap-environment/tutorial-overview?locale=en-US&state=PRODUCTION&version=Cloud) – ABAP env tutorials.
- [SAP BTP ABAP environment](https://help.sap.com/docs/cloud-identity-services/cloud-identity-services/sap-btp-abap-environment) – Entity reading and provisioning.

#### Chapter 7: Fiori & UI5 in BTP
- [Develop an SAP Fiori Application UI and Deploy it to ABAP Using SAP Business Application Studio](https://help.sap.com/docs/btp/sap-business-technology-platform/develop-sap-fiori-application-ui-and-deploy-it-to-abap-using-sap-business-application-studio) – Full overview.
- [Deployment of Application](https://help.sap.com/docs/bas/developing-sap-fiori-app-in-sap-business-application-studio/deployment-of-application) – Deployment troubleshooting.
- [Extend an SAP Fiori App and Deploy it to ABAP Using SAP BAS](https://help.sap.com/docs/sap-btp-abap-environment/abap-environment/extend-sap-fiori-app-and-deploy-it-to-abap-using-sap-bas) – Extension and deployment.
- [Creating an SAP Fiori App Using SAP Business Application Studio and Deploying It to SAP BTP Fiori Launchpad](https://help.sap.com/docs/SAP_EXTENSIBILITY_EXPLORER/a8b5387e2b4f42fd8b982ae8b4d02cde/0b2ace28fe704bb28e4af70adfcafa19.html) – Launchpad deployment.
- [Deploy the SAPUI5 Extension Project](https://help.sap.com/docs/bas/developing-sap-fiori-app-in-sap-business-application-studio/deploy-sapui5-extension-project) – Specific UI5 steps.

#### Chapter 8: Joule Fundamentals
- [Joule](https://help.sap.com/doc/75f5965fbb514fbcb89f1e47c094ea3f/CLOUD/en-US/9d8c0ef2443b49f993a27de8f38f83af.pdf) – Core fundamentals and conversational AI.
- [Joule in SAP SuccessFactors](https://help.sap.com/docs/joule/capabilities-guide/joule-in-sap-successfactors-4de4e7fb29434079ae07dffc93b64639) – Skills and agent capabilities.
- [AI Agent Extension to the Meta Model](https://help.sap.com/docs/leanix/ea/ai-agent-extension-to-meta-model) – Agent vs. skill concepts in AI context.
- [Uploading AI Agents Through the API](https://help.sap.com/docs/leanix/ea/discovering-ai-agents-using-api) – Basic agent building.
- [SAP Business Technology Platform](https://help.sap.com/doc/bd6250c40c9c4b60ae5e88c00c01486a/CLOUD/en-US/FSD_IntegrationSuite.pdf) – Ties Joule into BTP (broad).

#### Chapter 9: Building Your First Joule Skill
- [Set Up Joule Studio](https://help.sap.com/docs/Joule_Studio/45f9d2b8914b4f0ba731570ff9a85313/04b323352fa645238211ce017f634d34.html) – Booster setup for skills.
- [Integrating Joule with SAP Solutions](https://help.sap.com/doc/de3af3c0f81642dbaa4d36172ed57a72/CLOUD/en-US/79bfc83ab386450c8cd9c7937ce26a3a.pdf) – Destination creation and skills.
- [Enable Joule in the Cockpit](https://help.sap.com/docs/btp/sap-business-technology-platform/enable-joule-in-sap-btp-cockpit) – First steps.
- [Joule in SAP Fieldglass](https://help.sap.com/docs/SAP_Fieldglass/76e97fe0cccf4d37a5dadc652a375e33/e3dc12e0215447cfa4507eff66e8224b.html) – Permission and setup.
- [Joule](https://help.sap.com/doc/75f5965fbb514fbcb89f1e47c094ea3f/CLOUD/en-US/9d8c0ef2443b49f993a27de8f38f83af.pdf) – Directing to apps for skills.

#### Chapter 10: Building Joule Agents
- [Set Up Joule Studio](https://help.sap.com/docs/Joule_Studio/45f9d2b8914b4f0ba731570ff9a85313/04b323352fa645238211ce017f634d34.html) – Agent building basics.
- [Joule](https://help.sap.com/doc/75f5965fbb514fbcb89f1e47c094ea3f/CLOUD/en-US/9d8c0ef2443b49f993a27de8f38f83af.pdf) – Agent in BTP context.
- [Enable Joule in the Cockpit](https://help.sap.com/docs/btp/sap-business-technology-platform/enable-joule-in-sap-btp-cockpit) – Activation.
- [Integrating Joule with SAP Solutions](https://help.sap.com/doc/de3af3c0f81642dbaa4d36172ed57a72/CLOUD/en-US/79bfc83ab386450c8cd9c7937ce26a3a.pdf) – Subaccount setup for agents.
- [Prerequisites - Integrating Joule with SAP Solutions](https://help.sap.com/docs/joule/integrating-joule-with-sap/prerequisites) – Work Zone integration.

#### Chapter 11: Agent Lifecycle & Deployment
- [Manage Joule Skills](https://help.sap.com/docs/Joule_Studio/45f9d2b8914b4f0ba731570ff9a85313/ac5fd329d67844ae96697dec1bd77d22.html) – Deployment to test/prod.
- [Test a Joule Skill](https://help.sap.com/docs/Joule_Studio/45f9d2b8914b4f0ba731570ff9a85313/c4a47fa267ed442db7a822adc3da27f1.html) – Testing in lifecycle.
- [Deploy a Project - Joule Studio](https://help.sap.com/docs/Joule_Studio/45f9d2b8914b4f0ba731570ff9a85313/cb4c061a5d55492d80dd9204a32b74c1.html) – Deployment steps.
- [Transfer Projects Across Tenants - Joule Studio](https://help.sap.com/docs/Joule_Studio/45f9d2b8914b4f0ba731570ff9a85313/3cf6b29cc88845eab9e126f3a35e5548.html) – Dev to prod transfer.
- [2023 What's New for Joule Internal (Archive)](https://help.sap.com/docs/JOULE/4b327297dce247fcb88a5f5bfeea97a1/8cf77228a9e641e4be2f660ab70399f6.html?locale=en-US&state=PRODUCTION&version=INTERNAL) – CI/CD in lifecycle.

#### Chapter 12: Managing Multiple Clients on BTP
- [Using Multiple Subaccounts for Staged Application Development](https://help.sap.com/docs/btp/sap-btp-neo-environment/using-multiple-subaccounts-for-staged-application-development) – Isolation for multiple envs/clients.
- [Providing Multitenant Applications to Consumers in the Cloud Foundry Environment](https://help.sap.com/docs/btp/sap-business-technology-platform/providing-multitenant-applications-to-consumers-in-cloud-foundry-environment) – Subaccounts per consumer.
- [Adding and Managing Subaccounts](https://help.sap.com/docs/connectivity/sap-btp-connectivity-cf/managing-subaccounts) – Concurrent subaccount management.
- [Managing Users and Their Authorizations Using the btp CLI](https://help.sap.com/docs/btp/sap-btp-neo-environment/managing-users-and-their-authorizations-using-btp-cli) – Access for multi-client.
- [User Management](https://help.sap.com/docs/subscription-billing/setup-administration/user-management) – Users across subaccounts.

#### Chapter 13: Cross-Customer Deployments
- [Business Application Pattern](https://help.sap.com/docs/btp/sap-business-technology-platform/business-application-pattern) – Deployment patterns.
- [Cross-MTA Dependencies](https://help.sap.com/docs/btp/sap-business-technology-platform/cross-mta-dependencies) – Multi-MTA for cross-apps.
- [Developing Multitenant Applications in the Cloud Foundry Environment](https://help.sap.com/docs/btp/sap-business-technology-platform/developing-multitenant-applications-in-cloud-foundry-environment) – Multi-tenant patterns.
- [Catalog of Integration Use-Case Patterns](https://help.sap.com/docs/sap-btp-guidance-framework/sap-integration-solution-advisory-methodology/catalog-of-integration-use-case-patterns) – Cross-use patterns.
- [Multitenancy Development Guideline](https://help.sap.com/docs/sap-btp-abap-environment/abap-environment/multitenancy-development-guideline) – Tenant-aware apps.

#### Chapter 14: Building C-Level Agents for Enterprise Clients
- [Integrating Joule with SAP Solutions](https://help.sap.com/doc/de3af3c0f81642dbaa4d36172ed57a72/CLOUD/en-US/79bfc83ab386450c8cd9c7937ce26a3a.pdf) – Joule for enterprise subaccounts.
- [Joule Capabilities](https://help.sap.com/doc/1b82af8383e2443eaa95a034a70beb1b/CLOUD/en-US/c0bb884c3e27438695f4750b547aac77.pdf) – Advanced AI for C-level checks.
- [Phase 1. Discover: Achieve Architecture Transparency](https://help.sap.com/docs/leanix/ea/erp-transformation-discover) – Modeling for executives.
- [Understand General Rules for Advanced Formula Calculations for Planning](https://help.sap.com/docs/SAP_ANALYTICS_CLOUD/00f68c2e08b941f081002fd3691d86a7/766b9da1890d431ca29927daee4811b4.html) – Planning agents.
- [Operations Guide for SAP Digital Manufacturing](https://help.sap.com/doc/5796094639494b0d8524485c77e71a1b/latest/en-US/SAP_DMC_Operations_Guide_enUS.pdf) – Management-level ops.

#### Chapter 15: SAP Integration Suite Basics
- [Getting Started with SAP Integration Suite](https://help.sap.com/docs/integration-suite/sap-integration-suite/initial-setup) – Basics for flows and adapters.
- [What is SAP Integration Suite?](https://help.sap.com/docs/integration-suite/sap-integration-suite/what-is-sap-integration-suite) – Overview for hybrid scenarios.
- [Getting Started as a Developer in the ABAP Environment](https://help.sap.com/docs/btp/sap-business-technology-platform/getting-started-as-developer-in-abap-environment) – Ties to ABAP devs.
- [Developing with ABAP Cloud in the SAP BTP ABAP Environment](https://help.sap.com/docs/btp/btp-developers-guide/abap-cloud) – Integration in ABAP Cloud.
- [Feature Scope Description for SAP Integration Suite](https://help.sap.com/doc/e50e61e7b66c4b60ae5e88c00c01486a/CLOUD/en-US/FSD_IntegrationSuite.pdf) – Full scope.

#### Chapter 16: Cloud Connector for On-Premise Access
- [Configure Your On-Premise Systems to Use the Cloud Connector](https://help.sap.com/docs/SAP_ANALYTICS_CLOUD/00f68c2e08b941f081002fd3691d86a7/3bdb65253c8046b2b8234c554072569f.html) – Setup steps.
- [Connecting to On-Premise Systems Using Cloud Connector](https://help.sap.com/docs/migration-guide-po/migration-guide-for-sap-process-orchestration/connecting-on-premise-systems-cloud-connector) – Configuration in adapters.
- [Connect SCC to the On-Premise system](https://help.sap.com/docs/SUPPORT_CONTENT/fioritech/5173040117.html) – ABAP system connection.
- [Setting Up SAP Cloud Connector](https://help.sap.com/docs/cloud-portal-service/sap-cloud-portal-service-for-neo-environment/setting-up-sap-cloud-connector) – Admin setup.
- [Set Up the Cloud Connector (SAP HANA On Premise)](https://help.sap.com/docs/hana-cloud-database/sap-hana-cloud-sap-hana-database-data-access-guide/set-up-cloud-connector-sap-hana-on-premise) – Access control.

#### Chapter 17: Clean Core Rules That Bite Old Habits
- [Clean Core - SAP Build Process Automation](https://help.sap.com/docs/build-process-automation/sap-build-process-automation/clean-core) – Extensions without mods.
- [Finding and Exploring Classic APIs for Clean Core Development](https://help.sap.com/docs/abap-cloud/developer-guide-from-classic-abap-to-abap-cloud/finding-and-exploring-classic-apis-for-clean-core-development?locale=en-US&state=PRODUCTION&version=sap_cross_product_abap) – Wrappers for APIs.
- [Developer Guide: From Classic ABAP to ABAP Cloud](https://help.sap.com/docs/abap-cloud/developer-guide-from-classic-abap-to-abap-cloud/developer-guide-from-classic-abap-to-abap-cloud) – Transition steps.
- [Overview of the Key Concepts in ABAP Cloud](https://help.sap.com/docs/abap-cloud/developer-guide-from-classic-abap-to-abap-cloud/overview-of-key-concepts-in-abap-cloud?locale=en-US&state=PRODUCTION&version=sap_cross_product_abap) – Separation rules.
- [System View](https://help.sap.com/docs/cloud-alm/applicationhelp/system-view) – Clean Core checks.

#### Appendix A: Quick Reference Tables
- [Help Center for SAP Cloud Integration for data services](https://help.sap.com/doc/f710a11e51ac42d9bda8924f88d1121e/2509/en-US/hci10_help_sap_en.pdf) – Hierarchy examples.
- [Workflow in the Cloud Foundry Environment](https://help.sap.com/doc/01a7e4a41d9647728d0a7a55781c4033/Cloud/en-US/customer_workflow_cf_en.pdf) – Cheat sheets.
- [All Formulas and Calculations - SAP Analytics Cloud](https://help.sap.com/docs/SAP_ANALYTICS_CLOUD/00f68c2e08b941f081002fd3691d86a7/3b7c99a7a7cd4a29bb1c5132d91af341.html) – Reference tables.
- [Customize Date Dimensions - SAP Analytics Cloud](https://help.sap.com/docs/SAP_ANALYTICS_CLOUD/00f68c2e08b941f081002fd3691d86a7/6a22f9081ab540acb9e2cdbe4112ee11.html) – Hierarchy customization.
- [Workflow in the Neo Environment](https://help.sap.com/doc/4e57e50a863d45f9876d36e57dcc01ef/Cloud/en-US/customer_workflow_neo_en.pdf) – More cheat sheets.

#### Appendix B: Glossary for Old ABAPers
- [Glossary](https://help.sap.com/docs/btp/sap-business-technology-platform/btp-glossary) – BTP terms.
- [ABAP Glossary](https://help.sap.com/doc/abapdocu_cp_index_htm/CLOUD/en-US/ABENABAP_GLOSSARY.html) – ABAP-specific.
- [SAP Glossary](https://help.sap.com/glossary) – Full SAP terms.
- [ABAP Technology Glossary](https://help.sap.com/doc/abapdocu_cp_index_htm/CLOUD/en-US/ABENTECHNO_GLOSSARY.html) – Tech focus.
- [SAP Business Technology Platform](https://help.sap.com/doc/abapdocu_cp_index_htm/CLOUD/en-US/ABENSAP_BTP_GLOSRY.html) – BTP definition.

#### Appendix C: Useful Links & Resources
- [SAP Business Technology Platform](https://help.sap.com/docs/btp/sap-business-technology-platform/sap-business-technology-platform) – Onboarding and best practices.
- [Resources](https://help.sap.com/docs/btp/sap-business-technology-platform/resources) – MTA and resource types.
- [Environments - SAP BTP](https://help.sap.com/docs/btp/sap-business-technology-platform/environments) – PaaS offerings.
- [Basic Platform Concepts](https://help.sap.com/docs/btp/btp-admin-guide/basic-platform-concepts) – Core resources.
- [SAP Help Portal](https://help.sap.com/docs) – Main documentation hub.

#### Appendix D: Troubleshooting Common Issues
- [Troubleshooting - Joule](https://help.sap.com/docs/joule/integrating-joule-with-sap/troubleshooting) – Joule-specific issues.
- [Troubleshooting - Joule Studio](https://help.sap.com/docs/Joule_Studio/45f9d2b8914b4f0ba731570ff9a85313/f4cd9241a8e14f13ba960efc81501dd6.html) – Common solutions.
- [Troubleshooting Joule in the Cockpit](https://help.sap.com/docs/btp/sap-business-technology-platform/troubleshooting-joule-in-sap-btp-cockpit) – Subaccount/user errors.
- [Joule](https://help.sap.com/doc/75f5965fbb514fbcb89f1e47c094ea3f/CLOUD/en-US/9d8c0ef2443b49f993a27de8f38f83af.pdf) – Destination troubleshooting.
- [Troubleshooting for SAP Build](https://help.sap.com/docs/build-service/build-service-guide/troubleshooting-for-sap-build) – BTP build issues.