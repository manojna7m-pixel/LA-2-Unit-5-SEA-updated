# DevOps Practices and CI/CD Pipelines for Online Shopping Application

**Name:** Manojna M  
**USN:** NB25CSE204  
**Subject:** Software Engineering and Applications (Unit V)  
**Assigned Case Study:** Question 3 – DevOps Practices and CI/CD Pipelines  

---

## 1. Introduction

In a conventional software environment, development and operations teams often handled their work independently. Developers focused on writing application code, while the operations team took care of servers, testing, and releases. Because these activities were not closely coordinated, problems could take longer to identify and resolve.

**DevOps** brings these teams together through collaboration, automation, and shared responsibility. It supports the complete software lifecycle, including development, testing, release, and monitoring after deployment.

### Continuous Integration, Delivery, and Deployment

At the heart of modern DevOps are three core automation practices:

- **Continuous Integration (CI):** Developers regularly upload their code changes to a common repository such as GitHub. Every update can start an automated build and test process. Errors such as incorrect logic, invalid syntax, or database issues can therefore be detected early.
- **Continuous Delivery (CD):** After the code passes the required checks, it is packaged and moved to a staging environment similar to the live system. The application remains ready for release, while the final production deployment can still require approval from a responsible team member.
- **Continuous Deployment:** This approach takes automation one step further: every change that passes the defined tests is released to the production environment automatically, without a separate manual approval step.

---

## 2. Problem Analysis

The case study focuses on a company that operates an **online shopping application**. Such a platform needs to remain reliable and convenient for customers. Its development team may regularly introduce changes such as:
- New flash sale banner features and discount coupon calculators.
- Bug fixes for checkout crashes and cart calculation discrepancies.
- Third-party payment gateway integrations (UPI, credit cards, net banking).

### Existing Bottlenecks with Manual Workflows

1. **Slow Release Process:** Manually checking every feature takes considerable time. Testers may need to verify login, product search, cart operations, and checkout on different browsers. As more features are added, repeating these checks can delay important updates.
2. **Deployment Mistakes:** When releases depend on manual commands and configuration, small mistakes can affect the application. For example, an incorrect setting, missing dependency, or unsuitable database update may cause the deployed version to fail.
3. **Differences Between Environments:** An application may work correctly on a developer's computer but behave differently on the production server because software versions, libraries, or configuration settings do not match.
4. **Poor Customer Experience:** A faulty release can interrupt shopping activities. Customers may experience checkout problems, unavailable pages, or lost cart information, which can reduce trust in the service.

---

## 3. Proposed Solution: Automated CI/CD Pipeline

The company can reduce these problems by adopting an automated CI/CD pipeline. Tools such as Git, GitHub Actions or Jenkins, Docker, and Kubernetes can be used to manage the workflow.

### Pipeline Stages

The pipeline can be divided into the following stages, with each stage depending on the successful completion of the previous one:

1. **Source / Version Control:** Developers work on feature branches and push code to GitHub. Opening or merging a Pull Request into `main` automatically triggers the CI runner.
2. **Build Stage:** The automated server pulls the latest commit, downloads required dependencies, and compiles the shopping platform's services (e.g., auth, product catalog, payment service).
3. **Automated Testing Stage:** The build runs a battery of automated tests:
   - *Unit Tests:* Verify isolated functions (e.g., calculating item discounts, tax rounding, inventory decrement).
   - *Integration Tests:* Check interactions between modules (e.g., checking if the order service correctly communicates with the payment gateway mock and inventory database).
   - *End-to-End (E2E) / Smoke Tests:* Simulate a user logging in, adding a laptop to the cart, applying a promo code, and completing checkout.
4. **Packaging & Containerization:** If all tests pass, Docker creates an immutable container image tagging the exact commit hash. This image contains the application runtime, dependencies, and configuration templates.
5. **Staging Deployment:** The image is automatically pushed to a container registry and deployed to a staging cluster that mirrors live production. Automated smoke checks run against staging to confirm health.
6. **Production Deployment:** For an e-commerce site, Continuous Delivery with rolling updates (or blue-green deployment) is recommended. Kubernetes swaps old pods with new pods incrementally. If health checks detect an error, traffic immediately rolls back to the previous stable release without downtime.
7. **Monitoring & Feedback:** Tools like Prometheus, Grafana, and error trackers log response latencies and server error rates. If checkout failures spike, the team receives instant alerts, and the metrics feed directly into the next sprint backlog.

---

## 4. CI/CD Pipeline Architecture Diagram

```
 +----------------------------------------------------------------------------------+
 |                            CONTINUOUS INTEGRATION (CI)                           |
 +----------------------------------------------------------------------------------+
        |
        v
  [ Developer Push ] 
        | (Git commit to GitHub)
        v
  +------------------+
  | 1. BUILD STAGE   | ---> Pull dependencies, compile frontend & backend microservices
  +------------------+
        | (Build Success)
        v
  +------------------+      +--------------------+      +----------------------+
  | 2. TEST STAGE    | ---> | Unit Tests (JUnit) | ---> | Integration Tests    |
  +------------------+      +--------------------+      +----------------------+
        |                                                          |
        | (All tests pass)                                         v
        |                                               +----------------------+
        |                                               | E2E Checkout Flow    |
        |                                               +----------------------+
        v
  +------------------+
  | 3. PACKAGE STAGE | ---> Build Docker image & push to container registry
  +------------------+
        |
 +----------------------------------------------------------------------------------+
 |                       CONTINUOUS DELIVERY / DEPLOYMENT (CD)                      |
 +----------------------------------------------------------------------------------+
        |
        v
  +------------------+
  | 4. STAGING       | ---> Deploy container to staging server & run smoke tests
  +------------------+
        |
        v
  [ Approval Gate ]  ---> Manual sign-off or automated criteria for live release
        |
        v
  +------------------+
  | 5. PRODUCTION    | ---> Zero-downtime rolling deployment (Kubernetes)
  +------------------+
        |
        v
  +------------------+
  | 6. MONITORING    | ---> Prometheus / Grafana tracking live latency & errors
  +------------------+
        |
        +------- Feedback Loop (Logs & alerts sent back to developers) -------> [ Next Fix ]
```

---

## 5. Detailed Explanation of Build, Test, and Deployment

### A. Build
After a code update is submitted, the CI system prepares a clean build environment. It installs the required libraries, checks the code for basic quality issues, and builds the shopping website along with its backend services. 

*Shopping App Example:* Suppose a developer adds a new payment option to the checkout page. The build stage checks whether the updated code and its dependencies are valid. If the build fails, the developer is informed before the change moves to the next stage.

### B. Test
The testing stage checks whether the application behaves as expected. It can include several levels of automated testing:
- **Unit Testing:** Fast tests verifying individual algorithms (e.g., ensuring a "20% OFF" coupon caps out at the max discount amount).
- **Integration Testing:** Tests verifying API contracts (e.g., ensuring order creation triggers reservation of stock in the inventory database).
- **Security & Vulnerability Checks:** Scanners check third-party npm/pip packages for known vulnerabilities (CVEs) before releasing.

*Shopping App Example:* If a change causes the payment page to handle an invalid card expiry value incorrectly, an automated test can identify the problem before the update is released.

### C. Deployment
Deployment moves the verified application version to the live environment while aiming to keep the shopping service available. Docker and Kubernetes can support this process in the following ways:
- The exact container tested in staging is what runs in production, eliminating "works on my machine" issues.
- **Blue-Green or Rolling Updates:** Instead of shutting down the store for maintenance, new application pods are started and warmed up. Once they pass initial health checks, router traffic seamlessly switches over to the new version. If any unhandled exception occurs, the system automatically rolls back to the previous stable container.

---

## 6. Benefits and Challenges of Automation

### Key Benefits

1. **Faster Time-to-Market & Frequent Updates:** Instead of saving up fixes for a stressful monthly release, features and bug fixes can be deployed several times a day safely.
2. **Early Defect Detection:** Bugs are discovered within minutes of a commit. Fixing an error while the code is fresh in the developer's mind takes significantly less time and costs far less than fixing a production crash during a holiday sale.
3. **High Reliability & Zero-Downtime Releases:** Automated test suites and rolling deployments prevent broken code from affecting shoppers, ensuring high cart conversion rates and system availability.

### Real-World Challenges & Limitations

1. **Initial Setup and Maintenance Overhead:** Designing resilient pipeline scripts, containerizing microservices, and configuring secret management requires significant initial engineering effort. Flaky tests that randomly fail also need continuous maintenance.
2. **Database Migrations:** While stateless application code can easily be rolled back, database schema changes (e.g., dropping or altering columns in user order tables) require backwards-compatible migration strategies so active transactions don't fail during rolling updates.

---

## 7. Conclusion

For an online shopping application, relying only on manual testing and deployment can make releases slow and increase the chance of service interruptions. DevOps practices and a well-planned CI/CD pipeline help development and operations teams work together through a consistent, automated process. Automating building, testing, packaging, deployment, and monitoring allows the company to deliver improvements more efficiently while maintaining a dependable shopping experience.

---

### References
- Sommerville, Ian. *Software Engineering*, 9th/10th Edition (DevOps and Continuous Integration chapters).
- Lab Manual Unit V: Software Applications and Emerging Trends.
