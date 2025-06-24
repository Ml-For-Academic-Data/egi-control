graph TB
    %% Internet and External Access
    INTERNET["🌐 Internet"]
    
    %% AWS Infrastructure
    subgraph "AWS Cloud"
        subgraph "EC2 Instance (t3.medium)"
            subgraph "Ubuntu Server 24.04 LTS"
                %% Kubernetes Cluster
                subgraph "K3s Kubernetes Cluster"
                    %% ArgoCD Namespace
                    subgraph "Namespace: argocd"
                        ARGO_SERVER["🔄 argocd-server<br/>(NodePort)"]
                        ARGO_CONTROLLER["⚙️ argocd-application-controller"]
                        ARGO_REPO["📦 argocd-repo-server"]
                    end
                    
                    %% Development Environment
                    subgraph "Namespace: desarrollo"
                        DEV_AIRFLOW["🔄 airflow-webserver<br/>(NodePort 30001)"]
                        DEV_SCHEDULER["⏰ airflow-scheduler"]
                        DEV_MYSQL["🗄️ mysql<br/>(PVC: mysql-pvc)"]
                        DEV_FRONTEND["🎨 frontend<br/>(NGINX)"]
                    end
                    
                    %% Testing Environment
                    subgraph "Namespace: pruebas"
                        TEST_AIRFLOW["🔄 airflow-webserver<br/>(NodePort 30002)"]
                        TEST_SCHEDULER["⏰ airflow-scheduler"]
                        TEST_MYSQL["🗄️ mysql<br/>(PVC: mysql-pvc)"]
                        TEST_FRONTEND["🎨 frontend<br/>(NGINX)"]
                    end
                    
                    %% Production Environment
                    subgraph "Namespace: despliegue"
                        PROD_AIRFLOW["🔄 airflow-webserver<br/>(NodePort 30003)"]
                        PROD_SCHEDULER["⏰ airflow-scheduler"]
                        PROD_MYSQL["🗄️ mysql<br/>(PVC: mysql-pvc)"]
                        PROD_FRONTEND["🎨 frontend<br/>(NGINX)"]
                    end
                    
                    %% Shared Storage
                    STORAGE["💾 Local Storage<br/>(30 GiB EBS)"]
                end
                
                %% System Services
                SWAP["💿 Swap File (2GB)"]
                KUBECTL["⚙️ kubectl config"]
            end
        end
        
        %% Security Groups
        subgraph "Security Group"
            SSH_RULE["🔒 SSH (Port 22)<br/>From: Mi IP"]
            K8S_API["🔐 K8s API (Port 6443)<br/>From: Mi IP"]
            NODEPORT["🌐 NodePorts (30000-32767)<br/>From: 0.0.0.0/0"]
        end
    end
    
    %% External Services
    subgraph "External Services"
        GITHUB["👨‍💻 GitHub<br/>(SSH Deploy Keys)"]
        GHCR["📦 GitHub Container Registry<br/>(Image Pull)"]
    end
    
    %% Network Connections
    INTERNET --> SSH_RULE
    INTERNET --> NODEPORT
    INTERNET --> K8S_API
    
    %% SSH Access
    SSH_RULE --> KUBECTL
    K8S_API --> ARGO_SERVER
    
    %% Application Access
    NODEPORT --> DEV_AIRFLOW
    NODEPORT --> TEST_AIRFLOW
    NODEPORT --> PROD_AIRFLOW
    
    %% ArgoCD GitOps Flow
    ARGO_REPO -.-> GITHUB
    ARGO_CONTROLLER -.-> DEV_AIRFLOW
    ARGO_CONTROLLER -.-> TEST_AIRFLOW
    ARGO_CONTROLLER -.-> PROD_AIRFLOW
    ARGO_CONTROLLER -.-> DEV_FRONTEND
    ARGO_CONTROLLER -.-> TEST_FRONTEND
    ARGO_CONTROLLER -.-> PROD_FRONTEND
    
    %% Image Pull
    GHCR -.-> DEV_AIRFLOW
    GHCR -.-> TEST_AIRFLOW
    GHCR -.-> PROD_AIRFLOW
    GHCR -.-> DEV_FRONTEND
    GHCR -.-> TEST_FRONTEND
    GHCR -.-> PROD_FRONTEND
    
    %% Storage Connections
    STORAGE --> DEV_MYSQL
    STORAGE --> TEST_MYSQL
    STORAGE --> PROD_MYSQL
    
    %% Internal Service Communications
    DEV_SCHEDULER -.-> DEV_MYSQL
    TEST_SCHEDULER -.-> TEST_MYSQL
    PROD_SCHEDULER -.-> PROD_MYSQL
    DEV_AIRFLOW -.-> DEV_MYSQL
    TEST_AIRFLOW -.-> TEST_MYSQL
    PROD_AIRFLOW -.-> PROD_MYSQL
    
    %% Resource Specifications
    subgraph "Recursos y Configuración"
        CPU["⚡ 2 vCPUs"]
        RAM["🧠 4 GB RAM"]
        DISK["💽 30 GiB Storage"]
        TIMEZONE["🕐 America/Argentina/Mendoza"]
    end
    
    %% Monitoring and Observability (Future)
    subgraph "Monitoreo (Futuro)"
        PROMETHEUS["📊 Prometheus"]
        GRAFANA["📈 Grafana"]
        LOGS["📝 Logs"]
    end
    
    %% Styling
    classDef awsStyle fill:#ff9900,stroke:#232f3e,stroke-width:2px,color:#fff
    classDef k8sStyle fill:#326ce5,stroke:#1a5490,stroke-width:2px,color:#fff
    classDef argoStyle fill:#ef7b4d,stroke:#c4621a,stroke-width:2px,color:#fff
    classDef appStyle fill:#28a745,stroke:#1e7e34,stroke-width:2px,color:#fff
    classDef storageStyle fill:#6f42c1,stroke:#563d7c,stroke-width:2px,color:#fff
    classDef securityStyle fill:#dc3545,stroke:#bd2130,stroke-width:2px,color:#fff
    classDef externalStyle fill:#17a2b8,stroke:#138496,stroke-width:2px,color:#fff
    classDef futureStyle fill:#6c757d,stroke:#545b62,stroke-width:2px,color:#fff
    
    class INTERNET,SSH_RULE,K8S_API,NODEPORT securityStyle
    class ARGO_SERVER,ARGO_CONTROLLER,ARGO_REPO argoStyle
    class DEV_AIRFLOW,DEV_SCHEDULER,DEV_FRONTEND,TEST_AIRFLOW,TEST_SCHEDULER,TEST_FRONTEND,PROD_AIRFLOW,PROD_SCHEDULER,PROD_FRONTEND appStyle
    class DEV_MYSQL,TEST_MYSQL,PROD_MYSQL,STORAGE storageStyle
    class GITHUB,GHCR externalStyle
    class PROMETHEUS,GRAFANA,LOGS futureStyle