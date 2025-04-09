# REST API Experiments 🧪

This repository provides an environment for benchmarking REST API implementations across multiple
programming languages and frameworks. Using Kubernetes as the orchestration platform, it allows for consistent,
reproducible performance testing and comparison between different technology stacks.

This environment can help you to decide which framework is suitable for a given project. Implement the testing
REST API endpoints that represent the business logic of your application, add it to the testing environment, implement 
tests simulating the user behaviour, and run the performance tests with custom amount of simultaneous connection.


## 🚀 Getting Started

### Prerequisites

Before you begin, ensure you have the following tools installed:

- [Docker](https://docs.docker.com/engine/install)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download)
- [K6](https://grafana.com/docs/k6/latest/set-up/install-k6/)

### Deploy using kubernetes with Minikube

1. **Start Minikube**
   ```bash
   minikube start
   ```

   To view the dashboard:
   ```bash
   minikube dashboard
   ```

2. **Deploy PostgreSQL**
   ```bash
   kubectl apply -f k8s-manifests/postgres/
   ```

3. **Restore the Database**

   With database running in the cluster, you can restore the database from the dump file. The dump file is located in the 
`database` directory. For ease of use, a bash script is provided to restore the database. Script initialises the database
to state with no data
   ```bash
   # This can be replaced by initialisation of your custom database
   chmod +x database/restore_db.sh  # (Optional) Set execute permissions
   ./database/restore_db.sh
   ```

4. **Configure Minikube to Use Local Docker Registry**
   ```bash
   eval $(minikube docker-env)
   ```

5. **Build & Deploy Experiment Images**

   Build all REST API implementation images:

   ```bash
   # Ktor
   docker build -t ktor-experiment ./ktor/

   # Spring Boot
   docker build -t spring-boot-experiment ./spring-boot/
   
   # Gin
   docker build -t gin-experiment ./gin/
   
   # ASP.NET Core
   docker build -t dotnet-experiment ./dotnet/
   
   # Flask
   docker build -t flask-experiment ./flask/
   
   # Express
   docker build -t express-experiment ./express/
   ```

6. **Deploy All Components**
   ```bash
   kubectl apply -f ./k8s-manifests/ -R
   ```

## 📊 Monitoring Setup

### Setting Up Grafana

1. Login to Grafana with default credentials:
    - Username: `admin`
    - Password: `admin`
    - Default URL: http://192.168.49.2:30030/

2. Add Prometheus as a data source:
    - Default Prometheus URL: http://192.168.49.2:30010
    - [Follow the Grafana guide](https://grafana.com/docs/grafana/latest/datasources/)

3. Import the dashboard:
    - Import the JSON file: `grafana_dashboard.json`
    - Select Prometheus as the data source

### Example Prometheus Queries

```
# JVM memory usage (Ktor)
sum(jvm_memory_used_bytes{job="ktor-experiment"})

# CPU usage (Ktor)
system_cpu_usage{job="ktor-experiment"}

# Memory usage (Gin)
process_resident_memory_bytes{job="gin-experiment"}

# CPU usage rate (Gin)
rate(process_cpu_seconds_total{job="gin-experiment"}[30s])
```

For more information on queries, see
the [Prometheus documentation](https://prometheus.io/docs/prometheus/latest/querying/basics/).

## 🧪 Performance Testing

The project uses [K6](https://k6.io/) for performance testing. Test scripts are located in the `k6` folder. K6 script files are 
meant to be run using bash scripts located in the `k6/scripts` directory. Those scripts define what tests are run, what
APIs are tested (this is defined by ports of APIs), and how many virtual users are used for the tests.

All tests should be run from the `k6` directory.

**Tests all scenarios and all APIs with 1000 virtual users:**

```bash
./scripts/test_script.sh 1000
```
> ⚠️ **Note:** Complete test suite takes approximately 2 hours to finish.

**Tests complex select with ramping up users**

This test doesn't import data, it only tests the complex select API.

```bash
./scripts/ramp_up_test.sh
```

**Tests API break point**

This test doesn't import data, it only tests the complex select API.

```bash
./scripts/break_point_test.sh
```

> ⚠️ **Note:** After this test some API could be broken so minikube restart is recommended.

All test results are saved in the `k6/test-results` directory as HTML and csv files.

**Test modify**

To test individual experiments, modify the bash scripts files. K6 scripts to be run are defined in `K6_TEST_SCRIPTS`
variable. To modify what APIs are tested, modify the `PORTS` variable. The ports are defined in the k8s manifests.

## 🔌 Service Ports

When using Minikube, services are exposed on the following ports:

| Service      | Port  |
|--------------|-------|
| PostgreSQL   | 30432 |
| Prometheus   | 30010 |
| Grafana      | 30030 |
| Spring Boot  | 30808 |
| Gin          | 30809 |
| Ktor         | 30810 |
| ASP.NET Core | 30811 |
| Flask        | 30812 |
| Express      | 30813 |

## ⚠️ Known Issues

- When running in a VM with limited resources, some pods might fail to start. Either increase VM resources or deploy and
  test pods individually.

## 🧩 Adding New Experiments

1. **Implement** the experiment in your chosen technology
2. **Match the API** defined in `open-api-definition.json` for consistency
3. **Expose metrics endpoint** for Prometheus to scrape (e.g.,
   use [Micrometer](https://mvnrepository.com/artifact/io.micrometer/micrometer-registry-prometheus) for Spring Boot)
4. **Dockerize** the project
5. **Create K8s manifests** (use existing files as templates)
6. **Add the service** to Prometheus configuration in `k8s-manifests/prometheus/prometheus-config.yml`
7. **Update performance tests** in the K6 scripts with your service's NodePort
8. **Monitor API** performance using Grafana or Prometheus. Add pane to Grafana dashboard for your service

## 📋 Example REST API framework employments

| Framework    | Language   | Directory                    |
|--------------|------------|------------------------------|
| Spring Boot  | Java       | [`spring-boot`](spring-boot) |
| Gin          | Go         | [`gin`](gin)                 |
| Ktor         | Kotlin     | [`ktor`](ktor)               |
| ASP.NET Core | C#         | [`dotnet`](dotnet)           |
| Flask        | Python     | [`flask`](flask)             |
| Express      | JavaScript | [`express`](express)         |

## 📁 Repository Structure

```
Experiments
├── database - Database dump files for restoration
├── dotnet - C# ASP.NET Core project
├── express - NodeJS Express project
├── flask - Python Flask project
├── gin - Golang Gin project
├── k6 - Load testing resources
│   ├── scripts - Bash scripts for running tests
│   ├── test-results - Test results in HTML format
│   ├── break-point-test.js - Break point testing
│   ├── complex-test.js - Complex scenario testing
│   ├── device-test.js - Device scenario testing
│   ├── import-test.js - Import scenario testing
│   ├── load-test.js - Load testing for complex selects
│   ├── ramp-up-test.js - Ramp up testing
│   ├── sleep-test.js - Sleep testing
│   ├── spike-test.js - Spike testing for complex selects
│   └── test-data.zip - Test data for import tests
├── k8s-manifests - Kubernetes deployment files
│   ├── debug - PVC debugging deployment
│   ├── dotnet-experiment - ASP.NET Core deployment files
│   ├── express-experiment - Express deployment files
│   ├── flask-experiment - Flask deployment files
│   ├── gin-experiment - Gin deployment files
│   ├── grafana - Grafana deployment files
│   ├── ktor-experiment - Ktor deployment files
│   ├── postgres - PostgreSQL deployment files
│   ├── prometheus - Prometheus deployment files
│   └── spring-boot-experiment - Spring Boot deployment files
├── ktor - Kotlin Ktor project
├── postman - Postman collection for API testing
├── spring-boot - Java Spring Boot project
├── docker-compose.yml - Docker Compose configuration
├── open-api-definition.json - API definition for test scenarios
└── README.md - This documentation file
```

## Citation & Source Code

If you use this benchmark environment in your research or projects, please cite this repository.

```bibtex
@misc{rest-api-benchmark,
  author = {Ondrej Olsak, Marta Jaros, Matej Sauer, Jiri Jaros},
  title = {API KuBench: A Kubernetes-Based Environment for Standardized REST API Framework Performance Evaluation},
  year = {2025},
  publisher = {GitHub},
  url = {https://github.com/SC-FIT/API-KuBench}
}
```

Feel free to contribute to the project by submitting issues or pull requests!

