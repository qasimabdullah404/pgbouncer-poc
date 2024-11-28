# PgBouncer in Kubernetes (POC)

This document provides the steps to set up and test [PgBouncer](https://www.pgbouncer.org/) in a Kubernetes environment using Minikube. The purpose of this setup is to demonstrate the basic functionality of PgBouncer for connection pooling in a PostgreSQL cluster.

## Setup

### 1. Start Minikube with Calico CNI

Use the following command to start Minikube with the Calico CNI (Container Network Interface) plugin and allocate appropriate resources:

```bash
minikube start --cni=calico --network-plugin=cni --memory=4G --cpus=2
```

### 2. Generate the Combined Manifest Using Kustomize

Run the following command to generate a combined Kubernetes manifest (manifest.yaml) using Kustomize. This will combine all the necessary Kubernetes resources into one file, and then you can apply it:

```bash
kubectl kustomize . > manifest.yaml
kubectl apply -f manifest.yaml
```

This will create the necessary Kubernetes resources for PgBouncer and PostgreSQL.

## Test

## 1. Port-forward PgBouncer to Localhost

To test the PgBouncer connection, forward the PgBouncer service to your local machine on port 6432:

```bash
kubectl port-forward svc/pgbouncer-svc 6432:6432
```

## 2. Connect to PostgreSQL via PgBouncer

Once the port-forwarding is set up, use psql to connect to PostgreSQL through PgBouncer:

```bash
psql -h localhost -p 6432 -U postgres -d postgres -c "select version();"
```

This command connects to the PostgreSQL database via PgBouncer and runs a simple query to retrieve the version of the database.

## 3. Check Active/Idle Connections

You can view the active and idle PostgreSQL connections by running the following SQL query within the psql command line:

```bash
SELECT * FROM pg_stat_activity WHERE datname='postgres';
```

This will show you the current connections to the postgres database. PgBouncer manages these connections, pooling them to improve efficiency. The query will display columns such as:

* pid: The process ID of the connection.
* usename: The username connected to the database.
* state: The current state of the connection (e.g., active, idle).

## Auto Termination of Idle Connections

By default, idle connections will be terminated after 30 seconds. This is controlled by the server_idle_timeout parameter in the PgBouncer configuration. You can find this setting in the ConfigMap for PgBouncer (**5-pgbouncer-cm.yaml**).

For example, the server_idle_timeout is set as follows in the configuration:

```
server_idle_timeout = 30  # Auto-terminate idle connections after 30 seconds
```

This ensures that idle connections do not remain open indefinitely, improving the overall resource management of the database.

## Notes

1. **Credentials**: This is a proof-of-concept (POC) deployment, so credentials are not encrypted. For production use, it's highly recommended to secure credentials using Kubernetes Secrets or another encryption method.
2. **Throwaway Cluster**: This setup is intended for a throwaway cluster and not for production. If you're planning to use PgBouncer in production, make sure to follow security best practices, including securing communication and using encrypted credentials.

## Additional Resources

1. [PgBouncer Official Documentation](https://www.pgbouncer.org/)
2. [Minikube Documentation](https://minikube.sigs.k8s.io/docs/)
3. [Kustomize Documentation](https://kustomize.io/)