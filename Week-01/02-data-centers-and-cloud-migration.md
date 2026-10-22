# Data Centers and Cloud Migration

## 1. What is a data center?

A **data center** is a facility containing IT equipment used to run applications, store data, and connect systems. It usually includes servers, networking equipment, storage systems, power systems, and cooling.

A **server** is a computer or system that provides data, resources, or services to other computers over a network.

## 2. Main data-center components

### Compute (host components)
Compute provides processing capacity to run operating systems, applications, and workloads. Examples include physical servers and virtual machines.

### Storage (storage components)
Storage keeps data, files, application information, and backups.

### Network (fabric components)
Networking connects servers, storage, users, and other systems. It allows data to move between the components.

**Remember:** Compute runs workloads, storage keeps data, and networking connects the systems.

## 3. Traditional IT challenges

Organizations managing their own infrastructure may need to plan for:
- Purchasing or renting space for equipment.
- Electricity and cooling.
- Hardware maintenance and replacement.
- Time needed to add more capacity.
- Monitoring and operational support.
- Backup, disaster recovery, and business continuity.
- Physical risks such as fire, flooding, or power failure.

Cloud services can reduce some physical infrastructure responsibilities, but organizations must still plan for availability, backups, security, and disaster recovery.

## 4. CAPEX vs. OPEX

| Term | Meaning | Example |
|---|---|---|
| **CAPEX** — Capital Expenditure | Money spent on assets or equipment, often as a larger upfront investment | Buying a physical server |
| **OPEX** — Operating Expenditure | Ongoing expenses for operating a business or service | Paying a monthly cloud-service bill |

Cloud services are often paid for as operating expenses, but the actual cost model depends on the service and contract.

## 5. What is cloud migration?

**Cloud migration** is the process of moving data, applications, and IT workloads from an existing environment—such as an on-premises data center—to cloud infrastructure or cloud services.

Migration is not simply copying files. Teams need to assess applications, plan the target environment, protect data, test the result, and monitor the service after launch.

## 6. Typical cloud-migration stages

1. **Assess:** Inventory the current applications, data, servers, dependencies, risks, and costs.
2. **Plan:** Choose an approach, target services, schedule, responsibilities, and rollback plan.
3. **Prepare:** Configure the cloud environment, networking, identity, security, and monitoring.
4. **Migrate:** Move data and workloads using the selected migration method.
5. **Test:** Check functionality, performance, security, data integrity, and connectivity.
6. **Go live:** Direct users or production traffic to the migrated service when ready.
7. **Monitor and optimize:** Track reliability, performance, security, and cost, then make improvements.

**In short:**

`Assess → Plan → Prepare → Migrate → Test → Go Live → Monitor`

## 7. Example: moving a company website

A company first reviews its website and database. It then plans suitable cloud resources, prepares networking and security, moves the website and data, tests everything, switches users to the new environment, and continues monitoring performance.

## 8. Class example: telecom data-center services

Class notes mentioned a Jazz data-center example involving **4G, SMS, and VAS**. In telecommunications, **VAS** commonly means **Value-Added Services**. These are examples of services that may rely on data-center infrastructure; the precise architecture depends on the provider.

## Key takeaway

Cloud migration is a planned process. A successful migration must consider application dependencies, data protection, security, testing, availability, and ongoing costs.
