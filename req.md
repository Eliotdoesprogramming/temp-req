# DC Utilization Dashboard Requirements

## 1. Data Requirements

### 1.1 Infrastructure Platforms

#### 1.1.1 Physical Infrastructure

The most granular unit is a single server. Telemetry is instrumented with OpenManage Enterprise.

**Metrics**
- Physical CPU cores
- CPU utilization
- Physical memory
- Memory utilization
- Power draw (watts)

**Hierarchy**
- Server → Rack → OCS → Datacenter

#### 1.1.2 Virtualized Infrastructure

The most granular unit is a virtual machine. Telemetry is instrumented from VROPs.

**Metrics**
- vCPU allocation, utilization, and total capacity
- Memory allocation, utilization, and total capacity
- Disk allocation, utilization, and total capacity

**Hierarchy**
- Virtual Machine → Cluster → vCenter → Datacenter
- Each cluster can be mapped to its associated rack

#### 1.1.3 Containers on Baremetal Infrastructure

The most granular unit is a container.

**Metrics**
- vCPU allocation, utilization, and total capacity
- Memory allocation, utilization, and total capacity
- Disk allocation, utilization, and total capacity

**Hierarchy**
- Container → Namespace → Cluster → Foundation → Datacenter
- Clusters can be mapped to racks through CISPedia data

#### 1.1.4 Storage

TBD

### 1.2 Application & Lifecycle

The most granular unit is an Application, sourced from the product taxonomy in LeanIX.

#### 1.2.1 Vertical Lifecycles

- Strategy to Launch
- Market to Quote
- Order to Deliver Cash
- Plan to Manufacture
- Source to Pay
- Issue to Prevention
- Service Delivery Management
- Record to Report
- Plan to Actual

#### 1.2.2 Horizontal Lifecycles

- Artificial Intelligence
- Customer Data
- Dell IT
- Enterprise Data Management
- Facilities
- Legal
- Security & Resiliency
- Talent

#### 1.2.3 Application Data Requirements

- Lifecycle
- Application Owner
- L6-L3 Leader ownership
- Application Disposition (Northstar, set to retire)
- Launch date and planned retirement date (if applicable)
- Mapping of each granular unit of infrastructure resources to a particular application

### 1.3 Functional Data Requirements

- Data granularity: Daily with 90-day lookback period
- Data source: Data Science team's project database (metric store)
- A separate schema will be created to support the necessary views

---

### 1.4 Schema Definitions

The following schema definitions support the data requirements and UI components.

#### 1.4.1 Infrastructure Metrics Tables

##### physical_infrastructure_metrics

Daily metrics for physical servers.

| Column | Type | Description |
|--------|------|-------------|
| metric_date | DATE | Date of metric collection |
| server_id | VARCHAR (PK) | Unique server identifier |
| datacenter_id | VARCHAR | Datacenter identifier |
| ocs_id | VARCHAR | OCS identifier |
| rack_id | VARCHAR | Rack identifier |
| cpu_cores_total | INTEGER | Total physical CPU cores |
| cpu_utilization_pct | DECIMAL(5,2) | Average CPU utilization percentage |
| memory_total_gb | DECIMAL(10,2) | Total physical memory in GB |
| memory_utilization_pct | DECIMAL(5,2) | Average memory utilization percentage |
| power_draw_watts | DECIMAL(10,2) | Average power draw in watts |

##### virtual_infrastructure_metrics

Daily metrics for virtual machines.

| Column | Type | Description |
|--------|------|-------------|
| metric_date | DATE | Date of metric collection |
| vm_id | VARCHAR (PK) | Unique VM identifier |
| datacenter_id | VARCHAR | Datacenter identifier |
| vcenter_id | VARCHAR | vCenter identifier |
| cluster_id | VARCHAR | Cluster identifier |
| rack_id | VARCHAR | Rack identifier (mapped from cluster) |
| vcpu_allocated | INTEGER | Allocated vCPUs |
| vcpu_utilized | DECIMAL(10,2) | Average vCPUs utilized |
| vcpu_capacity | INTEGER | Total vCPU capacity |
| memory_allocated_gb | DECIMAL(10,2) | Allocated memory in GB |
| memory_utilized_gb | DECIMAL(10,2) | Average memory utilized in GB |
| memory_capacity_gb | DECIMAL(10,2) | Total memory capacity in GB |
| disk_allocated_gb | DECIMAL(10,2) | Allocated disk in GB |
| disk_utilized_gb | DECIMAL(10,2) | Average disk utilized in GB |
| disk_capacity_gb | DECIMAL(10,2) | Total disk capacity in GB |

##### container_infrastructure_metrics

Daily metrics for containers.

| Column | Type | Description |
|--------|------|-------------|
| metric_date | DATE | Date of metric collection |
| container_id | VARCHAR (PK) | Unique container identifier |
| datacenter_id | VARCHAR | Datacenter identifier |
| foundation_id | VARCHAR | Foundation identifier |
| cluster_id | VARCHAR | Cluster identifier |
| namespace | VARCHAR | Namespace identifier |
| rack_id | VARCHAR | Rack identifier (from CISPedia) |
| vcpu_allocated | DECIMAL(10,2) | Allocated vCPUs |
| vcpu_utilized | DECIMAL(10,2) | Average vCPUs utilized |
| vcpu_capacity | DECIMAL(10,2) | Total vCPU capacity |
| memory_allocated_gb | DECIMAL(10,2) | Allocated memory in GB |
| memory_utilized_gb | DECIMAL(10,2) | Average memory utilized in GB |
| memory_capacity_gb | DECIMAL(10,2) | Total memory capacity in GB |
| disk_allocated_gb | DECIMAL(10,2) | Allocated disk in GB |
| disk_utilized_gb | DECIMAL(10,2) | Average disk utilized in GB |
| disk_capacity_gb | DECIMAL(10,2) | Total disk capacity in GB |

#### 1.4.2 Application & Lifecycle Tables

##### application_master

Application metadata from LeanIX product taxonomy.

| Column | Type | Description |
|--------|------|-------------|
| application_id | VARCHAR (PK) | Unique application identifier |
| application_name | VARCHAR | Application name |
| lifecycle | VARCHAR | Associated lifecycle |
| application_owner | VARCHAR | Application owner name |
| leader_l6 | VARCHAR | L6 leader name |
| leader_l5 | VARCHAR | L5 leader name |
| leader_l4 | VARCHAR | L4 leader name |
| leader_l3 | VARCHAR | L3 leader name |
| disposition | VARCHAR | Application disposition (Northstar, Set to Retire) |
| launch_date | DATE | Application launch date |
| planned_retirement_date | DATE | Planned retirement date (nullable) |

##### infrastructure_application_mapping

Maps infrastructure resources to applications.

| Column | Type | Description |
|--------|------|-------------|
| application_id | VARCHAR (FK) | Application identifier |
| infrastructure_type | VARCHAR | Physical, Virtual, or Container |
| infrastructure_id | VARCHAR | Server, VM, or Container ID |

#### 1.4.3 Aggregated Views

Pre-aggregated views to support UI component performance.

##### vw_metric_summary_by_lifecycle

Aggregates metrics by lifecycle for lifecycle-focused components.

| Column | Type | Description |
|--------|------|-------------|
| metric_date | DATE | Date of metrics |
| lifecycle | VARCHAR | Lifecycle name |
| infrastructure_type | VARCHAR | Physical, Virtual, or Container |
| datacenter_id | VARCHAR | Datacenter identifier |
| count_infrastructure_units | INTEGER | Count of servers/VMs/containers |
| cpu_allocated | DECIMAL(15,2) | Sum of allocated CPU |
| cpu_utilized | DECIMAL(15,2) | Sum of utilized CPU |
| cpu_capacity | DECIMAL(15,2) | Sum of CPU capacity |
| memory_allocated_gb | DECIMAL(15,2) | Sum of allocated memory in GB |
| memory_utilized_gb | DECIMAL(15,2) | Sum of utilized memory in GB |
| memory_capacity_gb | DECIMAL(15,2) | Sum of memory capacity in GB |
| disk_allocated_gb | DECIMAL(15,2) | Sum of allocated disk in GB |
| disk_utilized_gb | DECIMAL(15,2) | Sum of utilized disk in GB |
| disk_capacity_gb | DECIMAL(15,2) | Sum of disk capacity in GB |
| power_draw_watts | DECIMAL(15,2) | Sum of power draw in watts (physical only) |

##### vw_application_metrics_summary

Aggregates metrics by application for application-focused tables.

| Column | Type | Description |
|--------|------|-------------|
| metric_date | DATE | Date of metrics |
| application_id | VARCHAR | Application identifier |
| application_name | VARCHAR | Application name |
| lifecycle | VARCHAR | Associated lifecycle |
| datacenter_id | VARCHAR | Datacenter identifier |
| vcpu_allocated | DECIMAL(15,2) | Sum of allocated vCPUs |
| vcpu_utilized | DECIMAL(15,2) | Sum of utilized vCPUs |
| memory_allocated_gb | DECIMAL(15,2) | Sum of allocated memory in GB |
| memory_utilized_gb | DECIMAL(15,2) | Sum of utilized memory in GB |
| disk_allocated_gb | DECIMAL(15,2) | Sum of allocated disk in GB |
| disk_utilized_gb | DECIMAL(15,2) | Sum of utilized disk in GB |

---

## 2. UI Requirements

### 2.1 Global Components

#### 2.1.1 Infrastructure Platform Filter

This component applies globally and allows filtering between infrastructure platforms.

**Behavior**
- Card components update according to the infrastructure platform selected
- When "All" is selected, components summarize multiple infrastructure platforms

#### 2.1.2 Location Filter

This component updates based on the infrastructure platform selected.

**Behavior**
- When "All" platforms is selected, defaults to Physical Infrastructure hierarchy
- Dropdown in breadcrumb format allows progressive depth selection

#### 2.1.3 Lifecycle Filter

Allows global filtering to a particular lifecycle.

**Behavior**
- Default value: All

### 2.2 Home Page

The home page consists of two main sections:
- Infrastructure Focused View (top section)
- Lifecycle Focused View (bottom section)

#### 2.2.1 Infrastructure Focused View

Displays data in an infrastructure-focused layout with top-down perspective.

##### Infrastructure Metric Cards

Per metric (Storage, Memory, CPU, Power), create cards with the following information:

**Data Elements**
- Total Capacity
- Allocation
- Average Utilization

**Behavior**
- Cards update based on global filters applied
- Hide cards if metric not applicable to selected infrastructure platform
- Utilization color-coded:
  - Red: Low utilization
  - Yellow: Medium utilization
  - Green: Efficient utilization
- Clicking card navigates to Metric Details Page

**Inputs**
- Location Filter selection
- Infrastructure Platform selection
- Lifecycle selection
- Per-metric data on current total capacity, allocation, and average utilization

**Page Location:** Home page, infrastructure focused view

##### Infrastructure Metric Trending Card

Per metric similar to the text metric cards above, the infrastructure metric trending card should show a summary of the trending information of the last 90 days.

**Data Elements**
- Minimum utilization (as line in timeseries chart)
- Maximum utilization (as line in timeseries chart)
- Average utilization (as line in timeseries chart)

**Behavior**
- Clicking card navigates to Infrastructure Metrics Detail Page for selected metric

**Inputs**
- Location Filter selection
- Infrastructure Platform selection
- Lifecycle selection
- Per-metric historical data of average utilization, allocation, and total capacity

**Page Location:** Home page, infrastructure focused view

#### 2.2.2 Lifecycle Focused View

Displays data with lifecycle-focused perspective, including lifecycle and application information with infrastructure resource summaries.

##### Lifecycle Radar Chart

Displays count of granular infrastructure units per lifecycle as a radar chart.

**Data Elements**
- Count of physical servers per lifecycle
- Count of VMs per lifecycle
- Count of containers per lifecycle

**Behavior**
- Hovering over a lifecycle displays tooltip with summation of each granular infrastructure unit

**Inputs**
- Location Filter selection
- Platform Filter selection

**Page Location:** Home page, lifecycle focused view

##### Lifecycle Summary Card

Per lifecycle, provides a summary card that works as an expandable drawer.

**Collapsed View**
- Count of granular infrastructure units (VMs, Containers, Physical Servers) associated with lifecycle

**Expanded View**
- Progress bars showing sums of average utilization vs. allocation for:
  - CPU
  - Memory
  - Disk
  - Power
- Top 5 applications (sorted by allocated memory), each showing:
  - Progress bars of utilization/allocation for CPU, Memory, Disk
- Link to Lifecycle Details Page

**Behavior**
- Progress bars display tooltip on hover showing detailed utilization/allocation information

**Inputs**
- Location Filter selection
- Platform Filter selection
- Per-lifecycle data on current utilization and allocation
- Count of granular infrastructure units associated with each lifecycle

**Page Location:** Home page, lifecycle focused view

### 2.3 Infrastructure Metrics Detail Page

Per metric, provides detailed view with the following components:

#### 2.3.1 Page Components

- Metric selected dropdown (pre-set to clicked metric)
- Lifecycle filter
- Location filter
- Platform filter
- Infrastructure metric card
- Infrastructure metric trending card
- Infrastructure lifecycle distribution chart
- Infrastructure application table

**Inputs:** Metric Selected

**Page Location:** Infrastructure Metrics Detail Page

#### 2.3.2 Infrastructure Lifecycle Distribution Chart

Pie chart showing lifecycle distribution by consumption of the selected metric.

**Behavior**
- Clicking lifecycle grouping updates lifecycle filter and applies to other page elements
- Hovering over lifecycle displays tooltip with allocated and utilized infrastructure information

**Inputs**
- Summation of current metric utilization per lifecycle
- Lifecycle filter selection
- Metric selected
- Location filter selection

**Page Location:** Infrastructure Metrics Details Page

#### 2.3.3 Infrastructure Applications Table

Paginated table of applications consuming infrastructure resources, sorted by total resources allocated.

**Columns**
- Application Name
- Lifecycle
- Progress bar comparing resources utilized to resources allocated (with tooltip on hover)

**Inputs**
- Metric selected
- Lifecycle filter selection
- Utilization and allocation details per application
- Location filter selection

**Page Location:** Infrastructure Metrics Details Page

### 2.4 Lifecycle Details Page

Application-focused page providing granular lifecycle information.

#### 2.4.1 Page Components

- Header showing lifecycle name and application count
- Infrastructure Metric Cards (CPU, Memory, Disk, Power) filtered to lifecycle
- Application Registry Table

**Inputs:** Selected Lifecycle

**Page Location:** Lifecycle Details Page

#### 2.4.2 Infrastructure Metric Cards

Cards showing utilized vs. allocated infrastructure per metric (CPU, Memory, Disk, Power).

**Inputs**
- Lifecycle
- Summary of average utilization and allocation associated with lifecycle

**Page Location:** Lifecycle details page

#### 2.4.3 Application Registry Table

Searchable table of applications associated with the lifecycle.

**Columns**
- Application Name
- vCPU utilized / vCPU allocated
- Memory utilized / Memory allocated
- Storage utilized / Storage allocated

**Behavior**
- Sortable by metric utilization
- Searchable by application name

**Inputs**
- Lifecycle
- Applications and summary of utilization and allocation per application

**Page Location:** Lifecycle details page
